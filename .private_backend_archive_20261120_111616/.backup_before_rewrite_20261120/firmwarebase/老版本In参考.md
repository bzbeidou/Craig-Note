#  CONV 指令集参考

---

## 1. 

| 术语 | 含义 |
|---|---|
| BD / BDC 引擎 | 张量计算引擎。CONV/MM/AR/RQDQ 等 14 类计算任务都由它执行 |
| NPU | Neural Processing Unit，计算核。本芯片 64 个，每个有独立 256KB 本地内存 |
| EU | Execution Unit，执行单元。每 NPU 16 个 32 位 EU（64 字节并行访存宽度） |
| LMEM | Local Memory，片上本地内存。64 NPU × 256KB = 16MB，按 NPU 分块 |
| OPD0~3 | 操作数 0~3（operand）。CONV 中：0=输入 1=权重 2=bias 3=pad/ins 值表 |
| RES0~1 | 结果 0~1（result）。CONV 中：0=输出 1=insert 常数值 |
| 命令字 / 描述符 | 一条硬件指令的编码形态（本指令 128B 长格式 / 64B 短格式） |
| 位域 | 把一个整数按位切给多个字段使用 |
| lane | 并行执行通道，一个 lane 对应一个 NPU |
| MAC | Multiply-Accumulate，乘加 |
| 加法树 | 多输入并行求和的硬件结构 |
| cycle | 硬件时钟周期，本模型下 1ns（1GHz） |
| TOPS / TFLOPS | 每秒万亿次（整数/浮点）运算 |
| tiling | 把大张量切成小块分多条指令计算 |

---

## 2. 指令标识与摘要

| 项目 | 值 | 出处 |
|---|---|---|
| 指令名 | CONV（卷积） | `bd_reg_value.h:10`（`TSK_TYPE` 枚举 `CONV=0`） |
| 所属引擎 | BD 引擎（ENGINE_BD=0） | `bm1684x/spec/include/engine_type.h` |
| 引擎命令内存基址 | `BDC_CMD_BASE_ADDR` | `bm1684x/spec/include/memmap.h:132` |
| 任务类型编码 | TSK_TYP = 0 | `bd_reg_value.h` |
| 指令长度 | 长格式 128 字节 / 短格式 64 字节 | `bd_reg_def.h` 末尾 `get_bdc_byte_len` |
| 寄存器视图 | `u32 cmd_buf[32]`（长格式） | `get_bdc_byte_len` 以 `cmd_buf[0]&1`、`cmd_buf[1]>>9` 解析 |
| 功能 | 带 bias 的二维卷积，支持空洞卷积、输入插洞、四边 padding、常数操作数、累加模式、核旋转 | 函数签名与实现 |



---

## 3. 指令格式

### 3.1 变长指令

BD 引擎的指令是**变长记录**，长度由两个因素决定：

1. **bit0（CMD_SHORT）**：0 = 长格式，1 = 短格式；
2. **任务类型（TSK_TYP）**：短格式下不同任务长度不同。

长度表（`get_bdc_byte_len`，`bd_reg_def.h`）：

| 任务类型 | TSK_TYP | 短格式长度 |
|---|---|---|
| SYS（系统） | 15 | 16 字节 |
| RQDQ / TRANS_BC / SFU / LIN | 4/5/9/10 | 32 字节 |
| MM（部分 EU 子类型）/ SG / CMP / VC | 2/6/13/14 | 32 / 48 字节 |
| **CONV / PD** | **0 / 1** | **64 字节** |
| AR（部分子类型） | 3 | 16 / 64 字节 |
| 任何任务，长格式 | — | 128 字节 |

### 3.2 长短格式的语义关系

- 短格式**不是**长格式的截断，而是**按任务类型重新紧凑打包**；
- CONV 的长短格式字段位置对照表在 `bm1684x/cmodel/include/short_cmd_map.h` 的 `CONV_map`；
- cmodel 执行时由 `cmd_mapping`（`cmodel_common.cpp:490`）把短格式展开成长格式；
  真机上由硬件直接识别短格式；
- 短格式丢掉的字段（取隐含默认值）：64 位 lane 掩码（隐含全 64 NPU 参与）、
  以及所有 CONV 不用的字段（详见 4.3）。

---

## 4. 编码格式

### 4.1 长格式总览（128 字节 = 32 个 32 位寄存器）

```
bit    0–63    命令头：格式标志/命令ID/依赖ID/任务类型/pad模式/ID使能/中断
bit   64–127   模式开关：累加/旋转/精度/常数标志/布局模式（含 25 位保留）
bit  128–191   卷积几何：插洞/空洞/四边padding/滑动步长/移位
bit  192–255   64 位 lane 掩码（CONV 恒为全 1）
bit  256–447   形状：RES0 输出 NCHW、OPD0 输入 CHW、OPD1 核 HW
bit  448–575   N/C 方向布局步长（16 位 × 8 个）
bit  576–703   四个 32 位地址：RES0/OPD0/OPD1/OPD2
bit  704–959   H/W 方向布局步长（32 位 × 8 个）
bit  960–1023  附加地址：RES1（insert 常数）/ OPD3（pad_ins 表）
```

### 4.2 长格式字段表（完整）

出处：字段名与位宽来自 `bm1684x/spec/include/bd_reg_def.h`；
"CONV 写入值"来自 `bm1684x/firmware_base/src/atomic/atomic_conv_gen_cmd.c:461`；
读取语义来自 `bm1684x/cmodel/src/atomic_conv.cpp:58`（`read_conv_reg`）。

| 寄存器 | 位 | 字段 | 宽度 | CONV 写入值 | 说明 |
|---|---|---|---|---|---|
| reg0 | 0 | CMD_SHORT | 1 | 0 | 0=长格式 |
| | 1–20 | CMD_ID | 20 | bd_cmd_id | 本命令编号 |
| | 21–31 | CMD_ID_DEP | 20 | gdma_cmd_id（低段） | 依赖的 GDMA 命令编号 |
| reg1 | 32–40 | CMD_ID_DEP | ↑ | （高段） | |
| | 41–44 | TSK_TYP | 4 | CONV(0) | 任务类型 |
| | 45–49 | TSK_EU_TYP | 5 | 不写（0） | 子类型；量化版写 1=普通 / 2=带ReLU |
| | 50 | EU_HALF_EN | 1 | — | |
| | 51–52 | TSK_OPD_NUM | 2 | — | |
| | 53–54 | PAD_MODE | 2 | pad_mode | 填充值模式 |
| | 55–58 | CMD_ID_EN | 4 | 1 | 启用编号/依赖机制 |
| | 59–62 | PWR_STEP | 4 | — | 电源步进 |
| | 63 | INTR_EN | 1 | — | 完成触发中断 |
| reg2 | 64 | OPT_RES_ADD | 1 | result_add | 累加模式（C=C+A·B） |
| | 65 | OPT_RELU | 1 | — | 量化版使用 |
| | 66–67 | LEFT_TRAN / RSVD1 | 2 | — | |
| | 68 | OPT_KERNEL_ROTATE | 1 | kernel_rotate | 卷积核旋转 180° |
| | 69–71 | OPT_OPD0/1/2_SIGN | 3 | — | 量化版写符号位 |
| | 72–74 | OPT_RES0_PREC | 3 | output_prec | 输出精度 |
| | 75–77 | OPT_OPD0_PREC | 3 | input_prec | 输入精度 |
| | 78–83 | OPT_OPD1/2_PREC | 6 | — | 量化版写权重/bias 精度 |
| | 84 | OPT_OPD0_CONST | 1 | — | 输入不能为常数 |
| | 85 | OPT_OPD1_CONST | 1 | kernel_is_const | 权重=常数立即数 |
| | 86 | OPT_OPD2_CONST | 1 | bias_is_const | 无 bias |
| | 87–89 | SHORT_RES0_STR | 3 | — | 输出布局固定 |
| | 90–92 | SHORT_OPD0_STR | 3 | input_short_str | 输入布局模式 |
| | 93–98 | SHORT_OPD1/2_STR | 6 | — | |
| reg3 | 99 | OPT_RES_ADD_SIGN | 1 | — | |
| | 100–125 | RSVD2 | 25 | — | 保留 |
| | 126 | OPT_OPD3_CONST | 1 | pad_ins_is_const | pad/ins 值=常数 |
| | 127 | RSVD4 | 1 | — | |
| reg4 | 128–131 | OPD0_X_INS0 | 4 | ins_w | 输入水平插洞数 |
| | 132–135 | OPD0_Y_INS0 | 4 | ins_h | 输入垂直插洞数 |
| | 136–139 | OPD1_X_INS0 | 4 | dilation_w − 1 | 核水平空洞（存值−1） |
| | 140–143 | OPD1_Y_INS0 | 4 | dilation_h − 1 | 核垂直空洞 |
| | 144–159 | OPD0_UP/DN/LF/RT_PAD | 4×4 | pad_h_t/b, pad_w_l/r | 四边填充 |
| reg5 | 160–163 | RES_OP_X_STR | 4 | stride_w | 水平滑动步长 |
| | 164–167 | RES_OP_Y_STR | 4 | stride_h | 垂直滑动步长 |
| | 168–191 | RES0/OPD0/OPD1 的 H/W_SHIFT | 4×6 | — | 其他任务用 |
| reg6–7 | 192–255 | TSK_LANE_NUM | 64 | 全 1 | 64 位 NPU 掩码 |
| reg8 | 256–271 | RES0_N | 16 | input_n | batch 数 |
| | 272–287 | RES0_C | 16 | output_c | 输出通道数 |
| reg9 | 288–319 | RES0_H / RES0_W | 16×2 | output_h / output_w | |
| reg10 | 320–335 | OPD0_N | 16 | 不写 | 输入 N 与输出 N 相同 |
| | 336–351 | OPD0_C | 16 | input_c | 输入通道数 |
| reg11 | 352–383 | OPD0_H / OPD0_W | 16×2 | input_h / input_w | |
| reg12 | 384–415 | OPD1_N / OPD1_C | 16×2 | 不写 | 权重形状可由 oc/ic/kh/kw 推导 |
| reg13 | 416–447 | OPD1_H / OPD1_W | 16×2 | kh / kw | 核尺寸 |
| reg14 | 448–479 | RES0_N/C_STR | 16×2 | 不写 | 输出布局固定对齐 |
| reg15 | 480–511 | OPD0_N_STR / OPD0_C_STR | 16×2 | str[0] / str[1] | 输入 N/C 步长 |
| reg16 | 512–543 | OPD1_N/C_STR | 16×2 | 不写 | 权重布局固定 |
| reg17 | 544–575 | OPD2_N/C_STR | 16×2 | 不写 | bias 布局固定 |
| reg18 | 576–607 | RES0_ADDR | 32 | output_addr | 输出地址 |
| reg19 | 608–639 | OPD0_ADDR | 32 | input_addr | 输入地址 |
| reg20 | 640–671 | OPD1_ADDR | 32 | weight_addr | 权重地址（或常数立即数） |
| reg21 | 672–703 | OPD2_ADDR | 32 | bias_addr | bias 地址（或常数） |
| reg22–23 | 704–767 | RES0_H/W_STR | 32×2 | 0 | |
| reg24–25 | 768–831 | OPD0_H_STR / OPD0_W_STR | 32×2 | str[2] / str[3] | 输入 H/W 步长 |
| reg26–27 | 832–895 | OPD1_H/W_STR | 32×2 | 0 | |
| reg28–29 | 896–959 | OPD2_H/W_STR | 32×2 | 0 | |
| reg30 | 960–991 | RES1_ADDR | 32 | ins_const_val | insert 常数值 |
| reg31 | 992–1023 | OPD3_ADDR | 32 | pad_ins_addr | pad/ins 值表地址（量化版：rshift_num） |

**自洽性说明**：生成代码中 `word5 high=0`、`word6 high=0`、`word7 low=0` 三个未写的
半字，正好对应表中 reg22–23、reg26–29 的"本指令不用"字段——写入与字段表完全对齐。

### 4.3 短格式（64 字节 = 512 位）

出处：`short_cmd_map.h` 的 `CONV_map`（长位 → 短位 的翻译表），
与 `atomic_conv_gen_cmd.c` FAST_GEN_CMD 分支逐项验证一致。

| 短格式位 | 宽度 | 字段 | 来源（长格式位） |
|---|---|---|---|
| 0 | 1 | CMD_SHORT = 1 | 0 |
| 1–20 | 20 | CMD_ID | 1（原位） |
| 21–40 | 20 | CMD_ID_DEP | 21（原位） |
| 41–44 | 4 | TSK_TYP | 41（原位） |
| 45–49 | 5 | TSK_EU_TYP | 45（原位） |
| 50–52 | 3 | OPD0/1/2_SIGN | 69/70/71 |
| 53–54 | 2 | PAD_MODE | 53（原位） |
| 55–58 | 4 | CMD_ID_EN | 55（原位） |
| 59–62 | 4 | PWR_STEP | 59（原位） |
| 63 | 1 | OPT_RES_ADD | 64 |
| 64 | 1 | OPT_KERNEL_ROTATE | 68 |
| 65–67 | 3 | OPT_RES0_PREC | 72 |
| 68–70 | 3 | OPT_OPD0_PREC | 75 |
| 71 | 1 | OPT_OPD1_CONST | 85 |
| 72 | 1 | OPT_OPD2_CONST | 86 |
| 73–76 | 4 | pad_h_t | 144 |
| 77–80 | 4 | pad_h_b | 148 |
| 81–84 | 4 | pad_w_l | 152 |
| 85–88 | 4 | pad_w_r | 156 |
| 89–92 | 4 | stride_w | 160 |
| 93–96 | 4 | stride_h | 164 |
| 97 | 1 | OPT_OPD3_CONST | 126 |
| 98–100 | 3 | SHORT_OPD0_STR | 90 |
| 101–126 | **26** | RES0_ADDR（输出地址，压缩到 26 位） | 576（32 位） |
| 127 | 1 | INTR_EN | 63 |
| 128–143 | 4×4 | ins_w / ins_h / dilation−1 | 128–143（原位） |
| 144–159 | 16 | OPD0_N_STR（str[0]） | 480 |
| 160–175 | 16 | RES0_N（input_n） | 256 |
| 176–191 | 16 | RES0_C（output_c） | 272 |
| 192–207 | 16 | RES0_H（output_h） | 288 |
| 208–223 | 16 | RES0_W（output_w） | 304 |
| 224–239 | 16 | OPD0_C（input_c） | 336 |
| 240–255 | 16 | OPD0_H（input_h） | 352 |
| 256–271 | 16 | OPD0_W（input_w） | 368 |
| 272–287 | 16 | OPD1_H（kh） | 416 |
| 288–303 | 16 | OPD1_W（kw） | 432 |
| 304–319 | 16 | OPD0_C_STR（str[1]） | 496 |
| 320–351 | 32 | OPD0_ADDR（input_addr） | 608 |
| 352–383 | 32 | OPD1_ADDR（weight_addr） | 640 |
| 384–415 | 32 | OPD2_ADDR（bias_addr） | 672 |
| 416–447 | 32 | RES1_ADDR（ins_const_val） | 960 |
| 448–479 | 32 | OPD3_ADDR（pad_ins_addr） | 992 |
| 480–495 | 16 | OPD0_H_STR（str[2]） | 768 |
| 496–511 | 16 | OPD0_W_STR（str[3]） | 800 |

**短格式丢弃的字段**（隐含默认值）：
- `TSK_LANE_NUM`（64 位）→ 隐含全部 64 个 NPU 参与。
  证据：`get_lane_mask`（`cmodel_common.cpp:1101`）对短命令直接返回全 1；
- OPT_RELU、符号位、OPD1/2 精度、6 个移位字段、OPD0_N、OPD1_N/C、
  RES0 各步长、OPD1/2 各步长 → 均为 CONV 不用的字段。

**设计哲学**：短格式 = 把"常见情况"固化为默认值，只编码与默认不同的部分。

---

## 5. 字段详细说明

### 5.1 命令控制字段

| 字段 | 说明 |
|---|---|
| CMD_SHORT | 长短格式标志。0=长（128B），1=短（64B） |
| CMD_ID | 本命令在 BD 引擎序列中的编号。每下发一条 BD 指令 +1（`cmd_id_proc.h`） |
| CMD_ID_DEP | 本命令依赖的**其他引擎**命令编号。CONV 通常填 GDMA 的当前编号 |
| CMD_ID_EN | 依赖机制使能。CONV 恒填 1 |
| INTR_EN | 指令完成后是否触发中断 |
| PWR_STEP | 电源步进（低功耗控制） |
| TSK_LANE_NUM | 64 位 lane 掩码，每位对应一个 NPU。CONV 恒为全 1（64 lane 全参与） |

### 5.2 任务与模式字段

**TSK_TYP**（任务类型，`bd_reg_value.h:9`）：

| 值 | 任务 | 值 | 任务 |
|---|---|---|---|
| 0 | **CONV（本文）** | 7 | LAR |
| 1 | PD（深度卷积） | 9 | SFU |
| 2 | MM（矩阵乘） | 10 | LIN |
| 3 | AR（张量算术） | 13 | CMP |
| 4 | RQDQ（量化） | 14 | VC |
| 5 | TRANS_BC | 15 | SYS（系统） |
| 6 | SG | | |

**TSK_EU_TYP**（EU 子类型）：浮点 CONV 不写（0）；量化 CONV 写 1=普通 / 2=带 ReLU。

**PAD_MODE**（`bd_reg_value.h:25`）：

| 值 | 模式 | 含义 |
|---|---|---|
| 0 | PAD_CONSTANT | 常数填充（最常用） |
| 1 | PAD_REFLECTION | 镜像填充 |
| 2 | PAD_REPLICATION | 复制边缘填充 |
| 3 | PAD_CIRCULAR | 循环填充 |

**PREC**（精度，`common/include/common.h:285`，非 ISA-V9 枚举）：

| 值 | 精度 | 值 | 精度 |
|---|---|---|---|
| 0 | INT8 | 5 | BFP16 |
| 1 | FP16 | 6 | INT4 |
| 2 | FP32 | 7 | FP8 |
| 3 | INT16 | 8 | FP20 |
| 4 | INT32 | 9 | TF32 |

**精度组合规则**（浮点 CONV，来自 ASSERT）：

| 输入精度 | 允许的输出精度 |
|---|---|
| FP16 | FP16 或 FP32 |
| BFP16 | BFP16 或 FP32 |
| FP32 | 仅 FP32 |

即：输出可与输入同精度，或升为 FP32（保留累加精度）；不允许降精度、不允许 FP16↔BFP16 混用。

**常数操作数标志**（OPT_OPDx_CONST）：置 1 时对应的 ADDR 字段不再当地址，
而是当作立即数，按当前精度解释（cmodel 中 `opd1_addr` 是 union：u32val/f32val/f16val）。
- OPD1_CONST=1：weight_addr 字段 = 常数权重的位串（标量卷积核）；
- OPD2_CONST=1：无 bias；
- OPD3_CONST=1：pad/ins 值用立即数（pad_ins_addr 当数值、RES1_ADDR 当 ins 值）。

### 5.3 几何字段

| 字段 | 范围 | 语义 |
|---|---|---|
| OPD0_X/Y_INS0（ins_w/h） | 0–14 | **输入侧插洞**：输入相邻像素间插入的空位数（作用于输入） |
| OPD1_X/Y_INS0（dilation_w/h） | 1–15（存值−1） | **核侧空洞**：卷积核元素间的间隔（作用于核） |
| OPD0_UP/DN/LF/RT_PAD | 0–15 | 输入上/下/左/右填充圈数 |
| RES_OP_X/Y_STR（stride_w/h） | 1–15 | 卷积核滑动步长（运算几何，注意与布局步长无关） |

### 5.4 形状字段

| 字段 | 语义 |
|---|---|
| RES0_N | batch 数 N（输入输出共用） |
| RES0_C / H / W | 输出通道数 / 高 / 宽 |
| OPD0_C / H / W | 输入通道数 / 高 / 宽（原始逻辑形状，不含 pad 与插洞） |
| OPD1_H / W | 卷积核高 / 宽 |

**输出尺寸由引擎按下面公式计算**（与生成侧 `atomic_conv_gen_cmd.c` 的推导一致）：

```
kh_ext  = dilation_h × (kh − 1) + 1
kw_ext  = dilation_w × (kw − 1) + 1
ih_ext  = (input_h − 1) × (ins_h + 1) + pad_h_t + pad_h_b + 1
iw_ext  = (input_w − 1) × (ins_w + 1) + pad_w_r + pad_w_l + 1
output_h = (ih_ext − kh_ext) / stride_h + 1
output_w = (iw_ext − kw_ext) / stride_w + 1
```

### 5.5 布局与步长字段

**SHORT_OPD0_STR（input_short_str，输入布局模式）**：

| 值 | 模式 | 含义 |
|---|---|---|
| 0 | align（默认） | 对齐布局，引擎自动计算步长 |
| 1 | compact | 紧凑布局 |
| 2 | 特殊 | n/h/w 步长=0，c 步长=1 |
| 3 | user | 使用指令中给出的显式步长 |

出处：`bm1684x/cmodel/include/cmodel_common.h:73` 注释及 `tensor_info_generate` 实现。

**默认（align）布局的步长计算**（`base_def.h:75`、`cmodel_common.cpp:144`）：

```
w_stride = 1
h_stride = W
c_stride = ALIGN(H × W, EU_NUM(精度))     // 对齐到 EU 宽度（元素数）
n_stride = ceil((C + 起始NPU) / 64) × c_stride
```

EU_NUM 按精度取：FP32→16、FP16/BFP16→32、INT8→64 个元素（恒等于 64 字节）。

**显式步长（short_str=3）**：OPD0_N/C/H/W_STR 分别给出 N/C/H/W 方向步长（单位：元素）。
生成侧有两条合法化简：
- input_n == 1 时 N 步长可置 0（只有一个 batch，永不需要走到下一个）；
- input_c + 起始NPU ≤ 64 时 C 步长可置 0（每个 NPU 至多一条通道，NPU 内偏移不前进）。

### 5.6 地址字段

地址为 **LMEM 地址**，编码方式：

```
地址 = NPU编号 << 18 | NPU内偏移        （LOCAL_MEM_SIZE = 256KB = 2^18）
```

| 字段 | 操作数 | 常数模式下的含义 |
|---|---|---|
| OPD0_ADDR | 输入特征图 | —（不可为常数） |
| OPD1_ADDR | 权重 | 常数权重的位串 |
| OPD2_ADDR | bias | 常数 bias 的位串 |
| OPD3_ADDR | pad/ins 值表 | pad 值的位串 |
| RES0_ADDR | 输出 | — |
| RES1_ADDR | insert 常数值 | — |

---

## 6. 操作数语义

| 泛化字段 | CONV 中的含义 | 形状 | 存放位置 |
|---|---|---|---|
| OPD0 | 输入特征图 | N × IC × IH × IW | 通道轮转分布于全部 NPU |
| OPD1 | 权重 | OC × ceil(IC/G) × KH × KW × G | 每个输出通道一组，放其所属 NPU |
| OPD2 | bias | OC（恒为 FP32） | 跟输出通道走 |
| OPD3 | pad/ins 值表 | IC × 2 | 跟输入通道走 |
| RES0 | 输出 | N × OC × OH × OW | 通道轮转分布于全部 NPU |
| RES1 | insert 常数值 | 标量 | 立即数 |

**OPD3（pad_ins 表）结构**：`[输入通道数][2]`——第 c 个输入通道两个相邻的值：
`[c][0]` = padding 填充值，`[c][1]` = insert 插入值（量化场景每通道零点不同，故按通道组织）。
出处：`atomic_conv.cpp:944`（FP32 路径 `(1, ic, 1, 2)` 的张量读取）。

**kernel_rotate**：置 1 时卷积核旋转 180°（配合 ins + stride=1 实现反卷积）。

**result_add**：置 1 时 `输出 = 原输出 + 本次卷积结果`（累加，不覆盖）。

---

## 7. 功能描述（伪代码）

```
输入先做"虚拟扩展"（不真实搬数据，仅寻址时体现）：
  对每个输入通道 c：
    在输入相邻像素之间插入 ins_h/ins_w 个空位，空位值为 ins_val[c]
    四边补 pad_h_t/b、pad_w_l/r 个空位，空位值为 pad_val[c]

然后对扩展后的输入做空洞卷积：

FOR n  = 0 .. N−1:                                  // 串行
  FOR oc = 0 .. OC−1:                               // 64 路并行（每 NPU 一条 lane）
    FOR (oh, ow) ∈ 输出像素, 每 4 个一批:            // 4 路并行
      acc = bias[oc] + (result_add ? RES0[n][oc][oh][ow] : 0)
      FOR kh = 0 .. KH−1, kw = 0 .. KW−1:           // 串行（核位置）
        FOR 每组 G 个输入通道:                        // G 路并行乘法
          p[g] = OPD0[扩展输入(通道 g)] × OPD1[oc][通道 g][kh][kw]
        acc = acc + ADD_TREE(p)                     // G 输入加法树归约
      RES0[n][oc][oh][ow] = 舍入(acc, output_prec)

其中 G（输入通道并行度）由精度决定：
  INT8 → 64，FP16/BFP16 → 32，FP32 → 1        （base_def.h:86 GET_CUBE_IC_PARALLEL）
```

cmodel 参考实现：`atomic_utils.cpp:159`（`conv_mac_binary16`）与 `:273`（`conv_core_float`）。

---

## 8. 存储器布局与寻址

### 8.1 通道分布规则（核心规则）

出处：`cmodel_common.cpp:188`（`gather_data`）。

```
张量起始 NPU 编号 S = 地址 >> 18，NPU 内偏移 = 地址 & (2^18 − 1)

通道 c 的元素 (n, c, h, w) 位于：
  NPU   = (S + c) mod 64
  偏移  = 基础偏移 + n×N_STR + floor((S+c)/64)×C_STR + h×H_STR + w×W_STR
```

即：**通道在 64 个 NPU 间轮转分布**；同一 NPU 内，每过 64 个通道偏移才前进一个 C_STR。
每个 NPU 的偏移不得超过其 256KB（`ASSERT(dst_offset < LOCAL_MEM_SIZE)`）。

### 8.2 同 NPU 归属规则

| 规则 | 原因 |
|---|---|
| 权重起始 NPU == 输出起始 NPU | 输出通道 oc 的计算发生在其所属 NPU，所需权重必须就地可取 |
| bias 起始 NPU == 输出起始 NPU | bias 按输出通道组织，同理 |
| pad_ins 起始 NPU == 输入起始 NPU | pad_ins 按输入通道组织，须与对应通道数据同 NPU |
| 输出起始 NPU | 任意 |
| 输入起始 NPU | FP32：任意；FP16/BFP16：必须为 NPU 0 |

最后一条的依据：非 FP32 时引擎按固定分组（FP16 每组 32 通道）并行读输入通道，
分组与 NPU 的对应从 NPU 0 开始固定接线；FP32 并行度为 1，地址通用。
**【推断】**：仓库无文档直述此硬件规定，此为依据布局与并行代码的推断；规则本身是 ASSERT 事实。

### 8.3 对齐要求

| 操作数 | 要求 | 说明 |
|---|---|---|
| 输入（默认布局） | 64 字节对齐 | 64 = ALIGN_BYTES = EU_NUM×4（`config.h:27`） |
| 输入（自定义步长） | 按元素对齐（FP16→2B，FP32→4B） | 布局由调用者全权负责 |
| 权重（非 FP32） | 64 字节对齐 | |
| bias | 4 字节对齐 | bias 恒为 FP32 |
| pad_ins 表 | 对齐到 2 个元素 | 表结构为 [通道][2] |
| 输出 | FP32→64 字节；其他→8 字节 | 8B = 4 个 16 位元素 = 每周期写出 4 个像素 |

---

## 9. 执行模型与性能

### 9.1 并行结构

```
每周期（FP16，处理 1 个核位置）：
  64 条输出通道 lane（每 NPU 一条） × 4 个输出像素 × 32 路输入通道乘法
  = 8192 次 MAC / 周期
```

| 精度 | 输入通道并行度 G | 像素并行 P | MAC/周期 | 峰值算力 @1GHz |
|---|---|---|---|---|
| INT8 | 64 | 4 | 16384 | 32.768 TOPS |
| FP16 / BFP16 | 32 | 4 | 8192 | 16.384 TFLOPS |
| FP32 | 1 | 16 | 1024 | 2.048 TFLOPS |

出处：`get_atomic_profile.cpp:746`（`atomic_conv_get_profile`）。
峰值算力与 BM1684X 公开标称（INT8 32 TOPS、FP16 16 TFLOPS）吻合，可作交叉验证。

### 9.2 周期数公式

```
cycle = N × ceil(OC/64) × ceil(OH·OW/P) × ceil(IC/G) × KH × KW
```

每个因子对应一个并行维度的分批次数：
- ceil(OC/64)：输出通道每 64 个一批（64 lane）；
- ceil(OH·OW/P)：输出像素每 P 个一批；
- ceil(IC/G)：输入通道每 G 个一批（加法树宽度）；
- KH×KW：**核面积串行**（一个核位置一个周期）；
- N：batch 串行。

### 9.3 数据流动

- 权重：每条 lane 从**本地** LMEM 每周期读一行（G 个值，权重平面按 G 宽行排布）；
- 输入：每周期 G×P 个值，从各通道所属 NPU 读出后**广播到全部 64 条 lane**；
- 输出：每 lane 每周期写 P 个像素，写回本地。

---

## 10. 合法性约束（违反 = 硬件未定义行为）

以下约束来自 `atomic_conv_gen_cmd.c:505` 起的 ASSERT 区（仅 cmodel 构建生效；
**真机上不检查，违反不会报错，只会算错或挂死**）。

### 10.1 值域

| 参数 | 范围 | 原因 |
|---|---|---|
| n/c/h/w, oc, kh, kw, ih_ext, iw_ext, oh, ow | 1 ~ 65535 | 16 位位域 |
| stride_h/w | 1 ~ 15 | 4 位位域，不可为 0 |
| pad（四边） | 0 ~ 15 | 4 位位域 |
| dilation_h/w | 1 ~ 15 | 4 位位域（存值−1） |
| ins_h/w | 0 ~ 14 | 4 位位域 |
| str[0..3] | 0 ~ 65535 | 短格式 H/W 步长仅 16 位 |
| 各布尔标志 | 0 或 1 | 1 位位域 |

注：ih_ext/iw_ext 是推导量，单独检查防止乘法溢出 16 位。

### 10.2 地址与对齐

见第 8.2、8.3 节。

### 10.3 精度组合

见第 5.2 节。

---

## 11. 依赖与同步

- 每条 BD 指令带 **CMD_ID**（自身编号）与 **CMD_ID_DEP**（依赖编号）；
- 典型用法：CONV 依赖某条 GDMA 指令（数据搬入 LMEM 完成后才能开始计算）；
- 硬件机制：引擎比较自身完成计数与指令中的依赖编号，满足才发射；
- 软件配套：`CMD_ID_NODE` 结构（`common/include/common.h:224`）维护各引擎计数器，
  `poll_all_engine_done` 通过读引擎寄存器计数确认全部完成。

---

## 12. 变体说明（量化 CONV，概要）

同文件中的量化变体复用同一描述符格式，差异（依据 `atomic_conv_quant_sym_gen_cmd`）：

| 差异点 | 说明 |
|---|---|
| TSK_EU_TYP | 写 1（普通）/ 2（带 ReLU） |
| 符号位（bit69–71） | 写 input/weight/bias 符号 |
| OPD1_PREC（bit78–80） | 写 INT8（权重精度） |
| OPD3_ADDR（reg31） | **字段复用**：存放 rshift_num（右移位数），而非 pad_ins_addr |
| OPD0_PREC | INT8 |
| 输出精度 | INT8 / INT16 |
| 输入起始 NPU | 必须为 NPU 0（`ASSERT(input_addr < LOCAL_MEM_SIZE)`） |

另有 `atomic_conv_gen_long_cmd`：与浮点 CONV 参数相同的长指令版本
（形状上限更大时使用，恒为 128 字节长格式）。

---

## 13. 编码示例

**设定**（FP16）：
输入 `1×32×8×8`，输出 64 通道，核 3×3，stride=1，pad=1，无插洞、无空洞，
带 bias，pad_ins 用常数，result_add=0，kernel_rotate=0，PAD_CONSTANT，
默认对齐布局（input_stride=NULL → short_str=0，str 全 0）。
输出尺寸 = 1×64×8×8。
地址假设：input=0x00000000，output=0x00800000，weight=0x01000000，bias=0x01800000。
命令编号：bd_cmd_id=1，依赖 gdma_cmd_id=5。

**长格式编码结果**（`u32 cmd_buf[32]`，逐寄存器）：

```
reg0  = 0x00A00002   // CMD_ID=1(bit1), CMD_ID_DEP=5(bit21)
reg1  = 0x00800000   // CMD_ID_EN=1 (bit55)
reg2  = 0x00000900   // RES0_PREC=FP16(1)<<8, OPD0_PREC=FP16(1)<<11
reg3  = 0x40000000   // OPD3_CONST=1 (bit126)
reg4  = 0x11110000   // 四边 pad 各=1 (bit16/20/24/28)
reg5  = 0x00000011   // stride_w=1(bit0), stride_h=1(bit4)
reg6  = 0xFFFFFFFF   \
reg7  = 0xFFFFFFFF   / lane 掩码全 1
reg8  = 0x00400001   // RES0_N=1, RES0_C=64
reg9  = 0x00080008   // RES0_H=8, RES0_W=8
reg10 = 0x00200000   // OPD0_C=32
reg11 = 0x00080008   // OPD0_H=8, OPD0_W=8
reg13 = 0x00030003   // OPD1_H=3(kh), OPD1_W=3(kw)
reg18 = 0x00800000   // RES0_ADDR（输出）
reg19 = 0x00000000   // OPD0_ADDR（输入）
reg20 = 0x01000000   // OPD1_ADDR（权重）
reg21 = 0x01800000   // OPD2_ADDR（bias）
reg12,14,15,16,17,22~31 = 0
```

**性能估算**（`atomic_conv_get_profile`）：

```
cycle     = 1 × ceil(64/64) × ceil(8·8/4) × ceil(32/32) × 3×3
          = 1 × 1 × 16 × 1 × 9 = 144 周期
op_count  = 1×64×8×8×3×3×32×2 = 2,359,296
利用率     = 2,359,296 / 144 = 16,384 op/周期 → 16.384 TFLOPS（满配峰值）
```

该例子各并行维度全部整除（OC=64、像素=64=16×4、IC=32=G），
是**算力利用率打满**的典型形状。

---

## 14. 源码对应索引

| 主题 | 文件 | 位置 |
|---|---|---|
| 指令生成（浮点 CONV） | `bm1684x/firmware_base/src/atomic/atomic_conv_gen_cmd.c` | :461 |
| 指令生成（量化 CONV） | 同上 | :5 / :229 |
| 字段位定义 | `bm1684x/spec/include/bd_reg_def.h` | 全文件 |
| 枚举值（TSK_TYPE/PAD_MODE 等） | `bm1684x/spec/include/bd_reg_value.h` | :9 / :25 |
| 精度枚举 | `common/include/common.h` | :285 |
| 硬件常量（NPU/EU/LMEM） | `bm1684x/spec/include/config.h`、`bm1684x/config_common.cmake` | |
| 通道并行度 | `bm1684x/spec/include/base_def.h` | :86 |
| cmodel 执行入口 | `bm1684x/cmodel/src/cmodel_common.cpp` | :547 execute_command |
| 字段读取（写读一致性） | `bm1684x/cmodel/src/atomic_conv.cpp` | :58 read_conv_reg |
| 内存布局模型 | `bm1684x/cmodel/src/cmodel_common.cpp` | :144 tensor_info_generate / :188 gather_data |
| 短格式翻译表 | `bm1684x/cmodel/include/short_cmd_map.h` | CONV_map |
| 短→长展开 | `bm1684x/cmodel/src/cmodel_common.cpp` | :490 cmd_mapping |
| 周期估算 | `bm1684x/cmodel/src/get_atomic_profile.cpp` | :746 |
| 指令长度解析 | `bm1684x/spec/include/bd_reg_def.h` | get_bdc_byte_len |
| 命令缓冲写入宏 | `bm1684x/firmware_base/include/gen_cmd.h` | WRITE_CMD_BD 等 |
| 命令编号/统计 | `bm1684x/firmware_base/include/cmd_id_proc.h` | :22 |
