## 第一部分：推导输出尺寸

```c++
 	int kh_ext = dilation_h * (kh - 1) + 1;
    int kw_ext = dilation_w * (kw - 1) + 1;
    int ih_ext = (input_h - 1) * (ins_h + 1) + pad_h_t + pad_h_b + 1;
    int iw_ext = (input_w - 1) * (ins_w + 1) + pad_w_r + pad_w_l + 1;
    int output_h = (ih_ext - kh_ext) / stride_h + 1;
    int output_w = (iw_ext - kw_ext) / stride_w + 1;
```

### 逐行解释

```
int kh_ext = dilation_h * (kh - 1) + 1;
```

**算“有效核高度”**：卷积核加上空洞后，实际覆盖了多少格。

- `kh` = kernel height，核高度；
- `dilation_h` = 空洞间隔；
- `_ext` = **effective**（有效的），即“等效尺寸”。

```
int ih_ext = (input_h - 1) * (ins_h + 1) + pad_h_t + pad_h_b + 1;
```

**算“等效输入高度”**：原始输入插洞、加上下 padding 之后的总高度。

```
int output_h = (ih_ext - kh_ext) / stride_h + 1;
```

**标准卷积输出尺寸公式**：`(等效输入 − 等效核) / 步长 + 1`。

注意一个**易混点**：这个函数里有两种完全不同的 "stride"：

- `stride_h / stride_w`：**卷积运算的滑动步长**（几何概念，上面公式用的）；
- `str[4] / input_stride`：**内存布局步长**（数据在内存里怎么排列，下一部分）。
	两者毫无关系，只是英文都叫 stride。

## 第二部分：决定输入的内存布局

```c++
    int input_short_str = input_stride == NULL ? 0 : 3;
    int str[4] = {0};
    if (input_stride != NULL) {
        memcpy(str, input_stride, 4 * sizeof(int));
        str[0] = input_n == 1 ? 0 : str[0];
        str[1] = (input_c + input_addr / LOCAL_MEM_SIZE) <= NPU_NUM ? 0 : str[1];
    }
```

```
int input_short_str = input_stride == NULL ? 0 : 3;
```

`input_short_str` 是**布局模式编号**，这是硬件和 cmodel 共同约定的一套编码（定义见 `bm1684x/cmodel/include/cmodel_common.h:73` 的注释）：

| 值   | 模式              | 含义                                         |
| ---- | ----------------- | -------------------------------------------- |
| 0    | **align（对齐）** | 默认布局，每行按 EU 对齐，**硬件自己算步长** |
| 1    | compact（紧凑）   | 不对齐、无缝排列                             |
| 2    | 特殊              | n/h/w 步长=0，c 步长=1                       |
| 3    | **自定义**        | 使用调用者传入的步长                         |

所以这行的意思是：**调用者没传 `input_stride`（NULL，空指针）→ 用默认对齐布局（0）；传了 → 用自定义布局（3）**。

```
int str[4] = {0};
if (input_stride != NULL) {
    memcpy(str, input_stride, 4 * sizeof(int));
```

把上层传来的步长数组拷进局部变量 `str`。`memcpy` 是内存拷贝函数（目标，源，字节数）。这个数组来自上层的 `dim4` 结构体，顺序是：

| 下标     | 含义                   |
| -------- | ---------------------- |
| `str[0]` | N 方向步长（batch 间） |
| `str[1]` | C 方向步长（通道间）   |
| `str[2]` | H 方向步长（行间）     |
| `str[3]` | W 方向步长（列间）     |

单位是**元素个数**（不是字节）。步长的意思：从当前元素走到“下一行/下一通道/下一个 batch”的地址，要跳过多少个元素。

### 接下来两条“强制置零”规则（最有含金量的部分）

```
str[0] = input_n == 1 ? 0 : str[0];
```

**batch 数为 1 时，N 步长没意义，直接置 0。**
道理很简单：只有一个 batch，永远不需要“走到下一个 batch”，这个步长根本不会被用到。步长=0 在硬件语义里就是“地址不前进”，完全安全。

```
str[1] = (input_c + input_addr / LOCAL_MEM_SIZE) <= NPU_NUM ? 0 : str[1];
```

**当所有输入通道都能落在不同的 NPU 上时，C 步长也可以置 0。**

这条需要理解 LMEM 的地址模型（我已从 `cmodel_common.cpp:1158` 的取地址代码验证）：

1. LMEM 地址的编码方式是 `NPU编号 << 每块大小 | NPU内偏移`，所以 `input_addr / LOCAL_MEM_SIZE` 就是**起始 NPU 编号**；
2. 输入通道是**按 NPU 轮转分布**的：第 c 个通道存在 NPU `(起始NPU + c) % NPU_NUM` 上（`%` 是取余）；
3. 只有当**两个通道落在同一个 NPU** 时，才需要用 C 步长在 NPU 内部跳地址。具体来说，通道 c 的 NPU 内偏移里有一项 `(起始NPU + c) / NPU_NUM * c步长`。

举例：`NPU_NUM = 64`，起始 NPU = 0：

```
input_c = 64  → 通道 0..63 分别落在 NPU 0..63，每个 NPU 只有 1 个通道
                → 没有任何两个通道共享 NPU → C 步长永远不会被用到 → 置 0 安全
input_c = 65  → 通道 64 绕回 NPU 0（第二个通道）
                → NPU 0 上有两个通道，必须用 C 步长区分 → 不能置 0
```

代码里用 `<=` 而不是 `<` 也是这个道理：`input_c + 起始NPU == 64` 时，最大通道编号是 63，落在 NPU 63 上，仍然没有共享。

**这两条本质上是“把用不到的步长归零”的优化**——不是改变行为，而是让调用者不必为一个不影响结果的字段填无意义的值。

## 这些推导结果去哪了

它们没有浪费，全部被写进了硬件命令字（就是函数后半段 `WRITE_CMD_BD(...)` 干的事）：

| 推导量                              | 用途                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| `output_h / output_w`               | 写进 word2 的 `RES0_H / RES0_W` 位域（告诉硬件输出多大）     |
| `input_short_str`                   | 写进 word0 的 `SHORT_OPD0_STR` 位域（告诉硬件输入用什么布局） |
| `str[0..3]`                         | 写进 word3/word6 的 `OPD0_N/C/H/W_STR` 位域                  |
| `ih_ext / iw_ext / kh_ext / kw_ext` | 除了算输出尺寸，还用于函数开头的合法性检查（ASSERT，断言）   |

一句话总结这段代码：**把“人类习惯的卷积描述”（形状、空洞、插洞、padding、步长）翻译成“硬件需要的描述”（输出尺寸 + 内存布局步长），并顺手做两处安全的化简。**

## 第三部分 `atomic_conv_gen_cmd` 的**参数合法性检查区（断言区）**

```c++
#ifdef USING_CMODEL
    ASSERT(input_addr < LOCAL_MEM_SIZE * (input_prec == FP32 ? NPU_NUM : 1));
    ASSERT(pad_ins_is_const || (pad_ins_addr / LOCAL_MEM_SIZE == input_addr / LOCAL_MEM_SIZE));
    ASSERT(kernel_is_const || (weight_addr / LOCAL_MEM_SIZE == output_addr / LOCAL_MEM_SIZE &&
                               weight_addr < LOCAL_MEM_SIZE * NPU_NUM));
    ASSERT(bias_is_const || (bias_addr / LOCAL_MEM_SIZE == output_addr / LOCAL_MEM_SIZE &&
                             bias_addr < LOCAL_MEM_SIZE * NPU_NUM));
    ASSERT(output_addr < LOCAL_MEM_SIZE * NPU_NUM);
    ASSERT((input_stride && (input_addr % get_bytesize(input_prec) == 0)) || (input_addr % 64 == 0));
    ASSERT((input_prec == FP16 && (output_prec == FP32 || output_prec == FP16)) ||
           (input_prec == BFP16 && (output_prec == FP32 || output_prec == BFP16)) ||
           (input_prec == FP32 && output_prec == FP32));
    ASSERT(kernel_is_const || (input_prec == FP32 ? 1 : (weight_addr % 64 == 0)));
    ASSERT(bias_is_const || bias_addr % sizeof(float) == 0);
    ASSERT(pad_ins_is_const || pad_ins_addr % (get_bytesize(input_prec) * 2) == 0);
    ASSERT(output_addr % (input_prec == FP32 ? 64 : 4 * 2) == 0);
    ASSERT(input_n < (((int)1) << 16) && (input_n > 0));
    ASSERT(input_c < (((int)1) << 16) && (input_c > 0));
    ASSERT(input_h < (((int)1) << 16) && (input_h > 0));
    ASSERT(input_w < (((int)1) << 16) && (input_w > 0));
    ASSERT(ih_ext < (((int)1) << 16) && (ih_ext > 0));
    ASSERT(iw_ext < (((int)1) << 16) && (iw_ext > 0));
    ASSERT(output_c < (((int)1) << 16) && (output_c > 0));
    ASSERT(output_h > 0 && output_h < (((int)1) << 16));
    ASSERT(output_w > 0 && output_w < (((int)1) << 16));
    ASSERT(stride_h > 0 && stride_h < 16);
    ASSERT(stride_w > 0 && stride_w < 16);
    ASSERT(pad_h_t >= 0 && pad_h_t < 16);
    ASSERT(pad_h_b >= 0 && pad_h_b < 16);
    ASSERT(pad_w_r >= 0 && pad_w_r < 16);
    ASSERT(pad_w_l >= 0 && pad_w_l < 16);
    ASSERT(dilation_h > 0 && dilation_h < 16);
    ASSERT(dilation_w > 0 && dilation_w < 16);
    ASSERT(ins_h >= 0 && ins_h < 15);
    ASSERT(ins_w >= 0 && ins_w < 15);
    ASSERT(kh > 0 && kh < 65536 && kw > 0 && kw < 65536);
    ASSERT(kernel_is_const >= 0 && kernel_is_const < 2);
    ASSERT(bias_is_const >= 0 && bias_is_const < 2);
    ASSERT(pad_ins_is_const >= 0 && pad_ins_is_const < 2);
    ASSERT(kernel_rotate >= 0 && kernel_rotate < 2);
#endif
```

在参数真正写进硬件命令字**之前**，把所有“硬件不允许的取值”提前拦下来。整段可以分成 5 组，每组背后都对应一条**真实的硬件规则**，不是随意写的软件偏好。

先解释这段代码里出现的几个基础名词：

| 名词                 | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| **ASSERT（断言）**   | 自检宏：条件不成立就打印错误并停住程序。定义在 `common/include/macros.h:120` |
| `x / LOCAL_MEM_SIZE` | 除以每块本地内存大小 → 得到 **NPU 编号**（因为地址编码是 `NPU号<<位宽 | NPU内偏移`） |
| `get_bytesize(prec)` | 返回一个元素占几个字节：INT8→1、FP16/BFP16→2、FP32→4（`common/include/common.h:314`） |

------

## 〇、先说最重要的一点：为什么包在 `#ifdef USING_CMODEL` 里

**这些检查只存在于 cmodel（软件仿真）版本，设备固件版本里这段代码完全不参与编译。**

断言失败时会发生什么（`common/include/macros.h:69`，cmodel 分支）：

```
printf("ERROR: ASSERT %s: %s: %d: %s\n", __FILE__, __func__, __LINE__, #_cond);
printf("ASSERT info: " fmt "\n", ...);
_print_trace();   // 打印调用栈：能看到是哪个算子、哪一层传错了参数
hang(-1);         // 把程序挂住停跑，保留出错现场
```

`#_cond` 里的 `#` 是“字符串化”操作——所以报错信息里会**原样打印失败的条件表达式**，比如 `ASSERT ... : input_addr % 64 == 0`，一眼就知道是哪条挂了。

**实践结论（很重要）**：

- 这些检查对应的是硬件约束（寄存器位宽、引擎寻址规则）。在 cmodel 上违反 → 立刻报错定位；**在真机上违反 → 不会有任何报错，直接算错结果或挂死**（未定义行为）。
- 所以“先在 cmodel 上跑通再上设备”不只是流程习惯，而是**唯一能抓住这类错误的机会**。

## 一、第 1~5 条：NPU 归属规则（“谁必须和谁在同一个 NPU”）

回忆一下内存模型：**LMEM（本地内存）按 NPU 分块**，bm1684x 每块 256KB、共 64 块；`地址 / LOCAL_MEM_SIZE` 就是 NPU 编号。

| 条   | 检查什么                                                     | 为什么                                                       |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | **输入**：非 FP32 必须从 NPU 0 开始；FP32 可以从任意 NPU 开始 | 卷积引擎对不同精度的**输入通道并行分组**不同：`base_def.h` 里 `GET_CUBE_IC_PARALLEL`（获取输入通道并行度）：INT8=64、FP16/BFP16=32、FP32=1。子 32 位精度下通道按固定分组铺到 NPU 上，起始位置被限制在 NPU 0；FP32 输入则把整个本地内存当连续空间用。（代码没写注释解释原因，这是依据引擎布局规则推断的） |
| 2    | **pad_ins 表**必须和**输入**在同一个 NPU                     | pad/insert 值是**按输入通道**查的数据，跟着输入走            |
| 3    | **权重**必须和**输出**在同一个 NPU，且在整个 LMEM 范围内     | 某个输出通道的计算就发生在它所在的那个 NPU 上，它要用的权重必须“就地”能取到 |
| 4    | **bias**必须和**输出**在同一个 NPU                           | bias 是**按输出通道**的数据，跟着输出走                      |
| 5    | **输出**可以在任意 NPU 开始                                  | 输出通道从起始 NPU 开始轮转分布，起点不限                    |

这里有个好记的规律：

> **权重和 bias 跟输出走（它们都是“每输出通道”的数据）；pad_ins 跟输入走（它是“每输入通道”的数据）。**

另外注意第 3、4 条都有 `xxx_is_const ||` 前缀——**常数模式没有地址**（值直接写进指令里），所以地址规则自然豁免。

------

## 二、第 6~11 条：对齐规则

“对齐”= 地址必须能被某个数整除。硬件按固定粒度访问内存，起始地址不对齐会导致取数错位。

| 条      | 规则                                                         | 原因                                                         |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 输入    | 默认布局：**64 字节**对齐；自定义 stride：只需**按元素**对齐（FP16→2B、FP32→4B） | 64 = `ALIGN_BYTES` = `EU_NUM × sizeof(float)` = 16×4，即 **EU（执行单元）的访存粒度**。默认“对齐布局”就是按这个粒度排的，起点当然也要落在这个边界上；紧凑/自定义布局由调用者全权负责，要求就放宽到元素边界 |
| 权重    | 非 FP32：64 字节对齐；FP32 豁免；const 豁免                  | 同上；FP32 走不同的布局路径                                  |
| bias    | 4 字节对齐（`sizeof(float)`）                                | **bias 恒为 FP32**（cmodel 里就是按 FP32 精度读它的），4B 是 float 的自然边界 |
| pad_ins | 对齐到 **2 个元素**（`元素大小×2`）                          | 这张表的结构是 `[输入通道数][2]`——每通道两个值（pad 值、ins 值）紧挨着存放，所以起点要对齐到“一对”的边界 |
| 输出    | FP32：64 字节；否则 8 字节（= 4 个 16 位元素）               | 引擎的写出粒度要求                                           |

------

## 三、第 12 条：精度组合规则

```
FP16  输入 → 输出只能是 FP16  或 FP32
BFP16 输入 → 输出只能是 BFP16 或 FP32
FP32  输入 → 输出只能是 FP32
```

翻译成人话，就是三条禁令：

1. **不能降精度**（FP32 进、FP16 出 ❌）；
2. **不能混格式**（FP16 进、BFP16 出 ❌）；
3. **允许升到 FP32**——这是刻意留的口子：卷积内部累加天然是高精度的，输出选 FP32 可以**保留累加精度**，做后续融合/高精度路径时有用。

------

## 四、第 13~26 条：数值范围（全部来自寄存器位域宽度）

回顾：**位域**（bitfield）= 把一个 64bit 整数按位切成小字段用。参数最终要塞进这些字段，**位域多宽，数值就只能多大**——这些断言就是在提前拦截“塞不下”的情况。

### 16 位位域（上限 65535 = 2¹⁶−1）

`input_n/c/h/w`、`output_c`、`kh`、`kw`、以及推导出来的 `output_h/output_w`，都必须 `> 0 且 < 65536`。

### 4 位位域（上限 15）

| 参数           | 范围 | 备注                                                         |
| -------------- | ---- | ------------------------------------------------------------ |
| `stride_h/w`   | 1~15 | 滑动步长不能为 0                                             |
| `pad_*`        | 0~15 | padding 可以为 0                                             |
| `dilation_h/w` | 1~15 | 空洞不能为 0（无空洞=1）                                     |
| `ins_h/w`      | 0~14 | ⚠️ 位域其实能存 15，代码收得略严一格，没有注释说明原因（可能预留） |

注意一个对照：**dilation 写入命令字时存的是“值−1”**（所以 1~15 对应位域里的 0~14），而 **ins 存原值**。

### `ih_ext / iw_ext` 为什么要单独检查？

这是个容易忽略的细节。它们是**乘法推导出来的量**：

```
ih_ext = (input_h - 1) * (ins_h + 1) + pad_h_t + pad_h_b + 1
```

即使 `input_h` 和 `ins_h` 各自都在合法范围内，**乘积也可能超出 16 位**。举个数：`input_h = 50000`（合法，<65536）、`ins_h = 14`（合法）→ `ih_ext ≈ 75万`，远超 65535。所以这两个推导量必须单独设卡。

------

## 五、最后 4 条：布尔标志

```
ASSERT(kernel_is_const  >= 0 && kernel_is_const  < 2);
ASSERT(bias_is_const    >= 0 && bias_is_const    < 2);
ASSERT(pad_ins_is_const >= 0 && pad_ins_is_const < 2);
ASSERT(kernel_rotate    >= 0 && kernel_rotate    < 2);
```

`>= 0 && < 2` 是“只能取 0 或 1”的标准写法。原因是这四个开关在命令字里各占 **1 个 bit**，传 2 就会污染相邻字段。

------

## 六、调试速查：这些断言挂了，先查什么

| 挂的断言                                   | 最可能的原因                                                 |
| ------------------------------------------ | ------------------------------------------------------------ |
| `input_addr < LOCAL_MEM_SIZE`（FP16 路径） | 输入张量起始地址没放在 NPU 0；通常是 LMEM 分配器（内存布局规划代码）给的偏移不对 |
| `weight_addr/NPU == output_addr/NPU` 不等  | 权重和输出分到了不同 NPU——搬数据（GDMA 拷入 LMEM）时的目标地址规划错了 |
| `xxx % 64 == 0`                            | 地址没按 64 字节对齐；常见于张量大小没向上取整到对齐边界就接着分配下一个 |
| `ih_ext < 65536`                           | 输入形状 × 插洞数太大，一条指令装不下 → 上层应该做切分（tiling，把大张量切成小块分多条指令算） |
| `stride_h < 16`                            | 上层传了超出硬件支持范围的卷积参数，需要软件层面等价变换或报不支持 |

一句话总结这一段：**它是“硬件说明书”的代码化——把寄存器位域宽度、引擎寻址规则、访存对齐要求全部翻译成可执行的检查，并且只在 cmodel 里生效，替你在仿真阶段就把真机上会静默出错的参数问题拦下来。**



## 第四部分  计算生成这条Conv需要的周期

我们推导并行结构用的那个 **cycle（时钟周期）公式，源头就是这一行**。它是把“这条卷积指令大概要跑多久”用纯软件公式**估算**出来并记录下来的钩子。

```c++
    CONV_GET_PROFILE(input_n, output_c, output_h, output_w, input_c, kh, kw, input_prec, pid_node);
```

```c++
#define CONV_GET_PROFILE(res_n, res_c, res_h, res_w, opdo_c, opd1_h, opd1_w, prec, pid_node)
#endif

#ifdef SG_STAS_GEN
#define SG_GET_PROFILE(res_n, res_c, opd1_w, opd3_addr, res_addr, sg_op,                         \
                         prec, pid_node)                                                         \
  if (pid_node != NULL) {                                                                        \
    pid_node->inst_profile = atomic_sg_get_profile(                                              \
        res_n, res_c, opd1_w, opd3_addr, sg_op, prec, res_addr);                                 \
    pid_node->cur_op_cycle = pid_node->inst_profile.cycle;                                       \
    strcpy(pid_node->cmd_name, "SG");                                                            \
  }
#else
```

```c++
INST_PROFILE atomic_sg_get_profile(
  int res_n,
  int res_c,
  int opd1_w,
  u32 opd3_addr,
  SG_OP op,
  PREC prec,
  u32 res_addr){
  UNUSED(prec);
  //int factor = 1;
  u64 cycle = 0;

  INST_PROFILE inst_profile;
  memset(&inst_profile, 0, sizeof(INST_PROFILE));
  int res_c_start = (int)(res_addr >> LOCAL_MEM_ADDRWIDTH);
  int c_per_npu = ceiling_func_shift(res_c_start + res_c, NPU_SHIFT);
  double op_count = (double)res_n *  res_c * opd1_w;

  switch (op)
  {
    case PE_S_gather_d1coor:
    case PE_S_scatter_d1coor:
    case PE_S_mask_select:
    case PE_S_nonzero:
    case PE_S_scatter_pp_d1coor:
    case PL_gather_d1coor:
    case PL_scatter_d1coor:
    case PL_gather_d2coor:
    case PL_scatter_d2coor:
        cycle = (u64)res_n * c_per_npu * opd1_w;
        break;
    case PL_gather_rec:{
        int kw = opd3_addr & 0xffff;
        int kh = opd3_addr >> 16;
        cycle = (u64)res_n * c_per_npu * opd1_w * kw * kh;
        break; }
    case PE_S_scatter_hzd:
    case PE_S_gather_hzd:
    case PE_S_nonzero_hzd:
        cycle = (u64)res_n * c_per_npu * opd1_w * 3 / 2;
        break;
    case PE_S_mask_selhzd:
        cycle = (u64)res_n * c_per_npu * opd1_w * 3;
        break;
    case PE_M_gather_d1coor:
        cycle = (u64)res_n * c_per_npu * opd1_w / 4;
        break;
    default:
      ASSERT_INFO(0, "sg_op (0x%d) \n", op);
      break;
  }
  cycle = (cycle==0)? 1: cycle;
  cycle = cycle * NCI;

  double compute_ability = (op_count * E10_6 * FREQ) / cycle;

  inst_profile.gdma_size = 0;
  inst_profile.cycle = cycle;
  inst_profile.compute_ability = compute_ability;

  return inst_profile;
}
```

------

## 一、先说清楚这行是干什么的

```
CONV_GET_PROFILE(input_n, output_c, output_h, output_w, input_c, kh, kw, INT8, pid_node);
```

**一句话**：用指令的形状参数，估算这条 CONV 指令在硬件上要消耗多少个时钟周期，把估算结果存进 `pid_node`（命令编号节点）。

三个关键性质：

1. **不生成任何指令内容**——它不影响写进命令字的任何一位；
2. **不影响计算结果**——纯粹是性能统计；
3. **默认编译配置下，这一行代码直接消失**（展开为空，编译器什么都不生成，零开销）。

------

## 二、名词解释

| 名词              | 含义                                                         |
| ----------------- | ------------------------------------------------------------ |
| **PROFILE**       | 性能剖析/画像：对一段代码或一条指令的耗时特征做测量或估算    |
| **STAS**          | statistics（统计）的缩写。`SG_STAS_GEN` = “统计生成”编译开关 |
| **cycle（周期）** | 硬件时钟周期，最小时序单位。`FREQ=1000`（MHz）→ 每周期 1ns（`get_atomic_profile.cpp:11` 的 `CLOCK_TIME`） |
| **TOPS**          | Tera Operations Per Second，每秒万亿次运算，芯片算力的常用单位 |
| **nJ**            | 纳焦耳，能耗单位（只在能耗估算分支用到）                     |

------

## 三、宏展开：两个分支（这是理解它的关键）

定义在 `bm1684x/firmware_base/include/stas_gen_macro.h:185`：

```
#ifdef SG_STAS_GEN
#define CONV_GET_PROFILE(res_n, res_c, res_h, res_w, opdo_c, opd1_h, opd1_w, prec, pid_node)  \
  if (pid_node != NULL) {                                                                      \
    pid_node->inst_profile = atomic_conv_get_profile(                                          \
        res_n, res_c, res_h, res_w, opdo_c, opd1_h, opd1_w, prec);                             \
    pid_node->cur_op_cycle  = pid_node->inst_profile.cycle;                                    \
    strcpy(pid_node->cmd_name, "CONV");                                                        \
  }
#else
#define CONV_GET_PROFILE(res_n, res_c, res_h, res_w, opdo_c, opd1_h, opd1_w, prec, pid_node)
#endif                                                                                         // ← 空！
```

而 `SG_STAS_GEN` 这个宏由 `options.cmake:178` 控制，**默认是 OFF**：

```
option(SG_STAS_GEN "option for stas gen" OFF)
```

所以分两种情况：

| 编译配置                  | 这行代码变成什么                                             |
| ------------------------- | ------------------------------------------------------------ |
| 默认（`SG_STAS_GEN=OFF`） | **什么都不生成**。普通的固件/cmodel 构建，追求性能，不带统计 |
| `SG_STAS_GEN=ON`          | 调用 `atomic_conv_get_profile(...)` 估算周期，存入 `pid_node`，并把命令标记为 `"CONV"` |

`if (pid_node != NULL)` 的保护表示：某些调用方不传统计节点（传 NULL）时也安全跳过。

------

## 四、参数逐个解释（宏的形参名已经写明了含义）

```
CONV_GET_PROFILE(input_n,   →  res_n    输出张量的 batch 数
                output_c,   →  res_c    输出通道数（每个输出通道一条 lane）
                output_h,   →  res_h    输出高
                output_w,   →  res_w    输出宽
                input_c,    →  opdo_c   输入通道数（OPD0=输入 的 C；宏里拼写少打了个 0）
                kh,         →  opd1_h   核高（OPD1=权重）
                kw,         →  opd1_w   核宽
                INT8,       →  prec     精度
                pid_node)   →  统计结果挂在哪个节点上
```

**注意一个细节**：你引用的这行来自 `atomic_conv_quant_sym_gen_cmd`（`atomic_conv_gen_cmd.c:111`，INT8 量化卷积），所以精度**硬编码**为 `INT8`——因为这个函数本身就是 INT8 专用的（它的断言就限定输出只能是 INT8/INT16）。对比一下文件里的 4 个调用点：

| 行号    | 所在函数                                 | 精度参数                        |
| ------- | ---------------------------------------- | ------------------------------- |
| **111** | `atomic_conv_quant_sym_gen_cmd`          | **INT8（硬编码）← 你问的这行**  |
| 341     | `atomic_conv_quant_asym_gen_cmd`         | INT8（硬编码）                  |
| 566     | `atomic_conv_gen_cmd`（浮点）            | `input_prec`（FP16/BFP16/FP32） |
| 785     | `atomic_conv_gen_long_cmd`（浮点长指令） | `input_prec`                    |

------

## 五、核心：`atomic_conv_get_profile` 内部算什么

实现在 `bm1684x/cmodel/src/get_atomic_profile.cpp:746`，就是我们前几轮反复用的那个公式：

```
u64 cycle = (u64)res_n * ceiling_func_shift(res_c, NPU_SHIFT)        // 输出通道：64 一批
                       * ceiling_func_shift(res_w * res_h, wh_shift) // 输出像素：4 一批
                       * opd0_c_cycle                                // 输入通道：64 一批(INT8)
                       * opd1_h * opd1_w;                            // 核面积：串行
cycle = cycle * NCI;                                                 // NCI = 1
```

其中 `ceiling_func_shift` 就是向上取整除法（`common.h:310`）：

```
INLINE static int ceiling_func_shift(int numerator, int shift) {
  return (numerator + (1 << shift) - 1) >> shift;    // = ceil(numerator / 2^shift)
}
```

对 INT8 代入常量（`NPU_SHIFT=6`、`EU_SHIFT_8BIT=6`、`wh_shift=2`、`NCI=1`）：

```
cycle = N × ceil(oc/64) × ceil(oh·ow/4) × ceil(ic/64) × kh × kw
```

**这正是前面推导“64 输出通道 × 4 像素 × 64 通道”并行结构的那个公式。** 换个角度看这行代码的意义：每生成一条 CONV 指令，就用它的形状参数过一遍这个公式，得到一个“这条指令要跑多少周期”的估算——公式的每个因子对应一个并行维度的分批次数。

除了 cycle，还算两个量：

```
double op_count = res_n * res_c * res_h * res_w * opd1_h * opd1_w * opd0_c * 2;
//                ↑ 总运算量。×2 = 乘法和加法各记一次（一次 MAC 算 2 个 op）

double compute_ability = (op_count * E10_6 * FREQ) / cycle;   // E10_6=1e-6, FREQ=1000
```

**`compute_ability` 的单位可以从常数推出来**：FREQ=1000 MHz → 每秒 10⁹ 周期 → 每秒运算数 = `op_count/cycle × 10⁹` → 换算成 TOPS（除以 10¹²）= `op_count/cycle × 10⁻³` = `op_count × 1e-6 × 1000 / cycle`。**正好就是上面的式子，所以单位是 TOPS**。

**用它反推峰值算力，可以交叉验证我们前面的并行模型**：一条“满配”指令（oc=64、像素=4、ic=64、核 1×1）每周期完成 `64 × 4 × 64 = 16384` 次 MAC = 32768 个 op，`compute_ability = 32768 × 1e-6 × 1000 / 1 = 32.768 TOPS`——与 BM1684X 公开的 INT8 峰值算力 **32 TOPS** 吻合；FP16 同理算出 **16.384 TFLOPS**，与公开的 FP16 **16 TFLOPS** 吻合。这从算力标称值的角度反向确认了前几轮推导的并行结构是正确的。

------

## 六、估算结果长什么样：`INST_PROFILE` 结构体

`common/include/common.h:203`：

```
typedef struct inst_profile {
  unsigned long long cycle;          // ← 核心：估算周期数
  unsigned long long gdma_size;      // GDMA 搬运量（CONV 恒为 0，搬运类指令才填）
  int gdma_direction;                // GDMA 方向
  int src_format, dst_format;        // 数据格式
  double op_dyn_energy;              // 动态能耗（nJ，仅 ENABLE_ENERGY 时有值）
  double sram_rw_energy;             // SRAM 读写能耗（nJ）
  double compute_ability;            // 算力利用率（TOPS）
  bool b_gdma_use_l2;
} INST_PROFILE;
```

------

## 七、数据流全景：估算出来的数字最终去哪

这是一条完整的统计链路（都在源码里验证过）：

```
① 生成指令时
   CONV_GET_PROFILE(...)                                [atomic_conv_gen_cmd.c:111]
     → pid_node->inst_profile = {cycle, compute_ability, ...}
     → pid_node->cur_op_cycle = cycle
     → pid_node->cmd_name = "CONV"

② 指令真正下发时（BEGIN_FAST_GEN_CMD 里的 id_proc 调到这里）
   BD_CMD_ID_PROC                                       [cmd_id_proc.h:22]
     → sg_stas_add_node(pid_node, ENGINE_BD)            // 记一条统计
     → pid_node->cycle_count += pid_node->cur_op_cycle  // 总周期累加

③ 一轮计算结束、等待引擎完成时
   poll_all_engine_done                                 [firmware_common_inline.h:236]
     → sg_stas_show(bd_cmd_id, gdma_cmd_id, cycle_count)
       → 写入统计文件，换算耗时和带宽利用率
```

`sg_stas_show` 里能看到最终输出形态（`sg_stas_gen_util.cpp:100`）：把 `cycle_count` 按每秒 10⁹ 周期换算成秒（`time_s = cycle_count / 1e9`），汇总 GDMA 搬运量算 DDR 带宽利用率，连同统计矩阵一起写进 `g_sg_multi_engine_stas_file` 文件。

------

## 八、数值例子（拿上一轮的例子代入）

设定：**INT8**，输入 `1×8×4×4`（8 通道，4×4 图），核 3×3，stride=1 无 pad，输出 4 通道、2×2：

```
cycle  = 1 × ceil(4/64) × ceil(2·2/4) × ceil(8/64) × 3 × 3
       = 1 ×    1       ×     1       ×    1       × 9
       = 9 个周期

op_count = 1×4×2×2 × 3×3 × 8 × 2 = 2304 个运算

compute_ability = 2304 × 1e-6 × 1000 / 9 ≈ 0.256 TOPS
```

**0.256 TOPS，只占了峰值 32.768 TOPS 的 0.8%。** 为什么这么低？对照并行结构就明白了：输出通道只用了 4/64 条 lane，输入通道只填了 8/64 个槽位，像素 4/4 倒是满了——**小张量根本喂不满并行阵列**。这也正是这个统计工具的价值：一眼看出“哪条指令的算力利用率低、为什么低”，为算子切分（tiling）和调优提供依据。

------

## 九、设计意图与局限

**为什么要这么设计：**

1. **零成本开关**——统计只在专门的 `SG_STAS_GEN` 构建里启用，正常固件/运行时构建完全不付出代价；
2. **不跑硬件就能估时**——纯解析公式（分析模型），生成指令的同时就知道它的周期代价，可用于编译期性能分析和调优决策；
3. **和指令生成共用同一套参数**——形状、核、精度本来就在手上，顺手估一次。

**局限（要知道它不是实测）：**

1. 这是**估算模型**，只算了计算部分的周期，没算内存带宽竞争、引擎间依赖等待等——真机实际耗时会更长；
2. 真正的**实测**走另一套机制：`END_FAST_GEN_CMD` 里的 `profile_time_begin/end`（固件里的计时打点），两者互补：一个“事前估算”，一个“事后实测”。
3. 能耗分支（`ENABLE_ENERGY`）只对 INT8 实现了，FP16/BFP16 还是 TODO（源码注释里写着 `// TODO for fp/bfp16`）。

------

## 十、总结

| 问题              | 答案                                                         |
| ----------------- | ------------------------------------------------------------ |
| 这行是干嘛的      | 性能统计钩子：估算这条 CONV 指令的硬件周期数、运算量、算力利用率，存进 `pid_node` |
| 它影响指令吗      | 不影响。不写命令字、不改计算结果                             |
| 默认编译下呢      | **整行消失**（`SG_STAS_GEN` 默认 OFF，宏展开为空）           |
| 为什么精度是 INT8 | 你看的这行在 `atomic_conv_quant_sym_gen_cmd`（111 行），INT8 量化卷积专用；浮点版本（566 行）传 `input_prec` |
| cycle 公式是啥    | `N × ceil(oc/64) × ceil(像素/4) × ceil(ic/64) × kh×kw`——就是前几轮推导并行结构的那个公式 |
| 数字最后去哪      | `pid_node` → 指令下发时累加进 `cycle_count` → `poll_all_engine_done` 时 `sg_stas_show` 写统计文件 |
| 怎么验证它靠谱    | 满配指令算出 32.768 TOPS (INT8) / 16.384 TFLOPS (FP16)，与 BM1684X 公开标称算力吻合 |

接下来函数就进入真正的 `BEGIN_FAST_GEN_CMD(BD)` / `WRITE_CMD_BD(...)` 段落——把参数逐位打包进命令字。那部分我们之前已经预演过位域映射，需要的话可以逐字（word0~word7）过一遍。