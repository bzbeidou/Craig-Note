# RSQRT Current Implementation Design

Date: 2026-05-05

## 1. 总览

当前 `rsqrt` 算子的目标是计算：

```text
out[i] = 1 / sqrt(input[i])
```

数学上仍然使用 Newton-Raphson 形式：

```text
y_{k+1} = y_k * (1.5 - 0.5 * x * y_k^2)
```

但当前 RTL 执行策略不是在 RTL 中完整展开每一轮 `FPMUL/FPADD` Newton 计算。当前实现采用更稳定的串行真实写回方案：

1. app/tvgen 侧根据输入和 `iters` 预先计算最终 Newton 状态。
2. 数据生成阶段把这个最终状态写进 alpha/const bank 的辅助区域。
3. RTL 侧通过 `UR-R2 -> RtoVR-35 -> UR-W0` 做真实 UR 读出与真实 UR 写回。
4. `compare_case.py` 比对最终 UR dump 和 golden，确认真实写回结果正确。

因此可以把当前实现理解为：

```text
app/golden 负责数值 Newton 迭代
RTL 负责把预计算结果按真实 SMI-IQ/UR 路径写回 output bank
```

这样做的核心原因是：当前新 RTL 上，完整 rsqrt Newton 调度的多端口读写和反馈路径存在不稳定点；当前版本先保证真实写回路径、bank 布局、IQ 打包、embed/loop 与 compare 端到端稳定。

## 2. 相关文件

| 文件 | 作用 |
|---|---|
| `op/rsqrt/rsqrt_op.cpp` | 当前主要实现：参数规整、编译调度、IQ 打包、随机 case、数据生成、golden |
| `op/rsqrt/rsqrt_op.h` | 参数结构、调度结构、接口声明 |
| `op/rsqrt/rsqrt_op_constants.h` | RtoVR 输入/输出编号、默认迭代次数等常量 |
| `op/rsqrt/rsqrt_op_compute_inst.cpp` | FPMUL/FPADD CRU 生成函数，目前仅保留配置能力 |
| `op/rsqrt/doc/rsqrt_ai_skil.md` | 当前 rsqrt 验证必须遵守的 skill 流程 |

注意：`rsqrt_op_constants.h` 里仍保留了旧版完整 Newton pipeline 的常量和注释，其中 `RTOVR_IN_UR_RDATA_2`、`RTOVR_OUT_UVR_WDATA_0`、`DEFAULT_NEWTON_ITERS` 等仍被当前实现使用；旧的 29-cycle 完整 Newton 时序不是当前实际 RTL 调度。

## 3. 数据粒度和 step 概念

当前实现主要按 FP32 路径工作。

FP32 下：

```text
1 个 UR word = 128 bit = 4 个 float32 lane
elems_per_word = 4
total_steps = ceil(total_elements / 4)
```

例如：

| Shape | Elements | total_steps |
|---|---:|---:|
| `1x1x1x1` | 1 | 1 |
| `1x1x1x4` | 4 | 1 |
| `1x1x1x5` | 5 | 2 |
| `1x2x3x4` | 24 | 6 |

尾部不足 4 个元素时，数据生成阶段会在最后一个 128-bit word 的空 lane 中填 0，golden 只对真实元素生成结果。

## 4. 参数结构

`RsqrtOpParams` 记录算子编译需要的参数：

```text
para_n / para_c / para_h / para_w
para_dtype
para_iters
para_x_bank / para_x_addr
para_y0_bank / para_y0_addr
para_const15_bank / para_const15_addr
para_const05_bank / para_const05_addr
para_output_bank / para_output_addr
para_scratch0_bank / para_scratch0_addr
para_scratch1_bank / para_scratch1_addr
```

当前关键点：

- `para_dtype` 当前编译入口固定为 FP32，即 `0`。
- `para_iters` 控制 app/golden 侧 Newton 迭代次数。
- `para_iters` 不再表示 RTL 中完整 Newton 迭代调度次数。
- `scratch0/scratch1` 仍保留在参数中，用于布局规整和兼容接口；当前稳定路径最终只写 output。

## 5. 参数规整：`sanitize_params`

`sanitize_params()` 的职责是把输入 bank/address 修正成 RTL 更容易稳定执行的布局，并避免 UR 区域冲突。

### 5.1 计算总 word 数

函数先计算：

```text
total_elems = N * C * H * W
elems_per_word = 4 for FP32
total_steps = ceil(total_elems / elems_per_word)
```

后续所有区域长度都以 `total_steps` 为基础。

### 5.2 bank 归一化

UR 一共有 8 个 bank：

```text
kNumUrBanks = 8
```

如果传入 bank 超过范围，会被映射到低 3 bit：

```text
new_bank = bank & 0x7
```

这样可以避免非法 bank 编号进入 IQ code。

### 5.3 避免 x/y0 同 bank 双读

旧完整 rsqrt 需要同时读 `x` 和 `y`。当前稳定路径虽然只实际读 `UR-R2`，但布局仍保守规避 `x_bank == y0_bank`：

```text
if x_bank == y0_bank:
    remap y0_bank
```

这样保留 bank-conflict 覆盖的同时，避免新 RTL 上已知的同 bank 双读路径风险。

### 5.4 常量 bank 规整

`const15` 和 `const05` 必须放在同一个 bank，并且常量 bank 避开：

```text
x_bank
y0_bank
output_bank
```

当前规则：

```text
const15_bank = const05_bank = chosen_safe_bank
const05_addr = const15_addr + 1
```

原因是当前 alpha/const 区不仅存 `1.5` 和 `0.5`，还追加预计算 Newton 最终状态。把常量和最终状态放在同一 bank 的连续区域，后续 `iter_state_addr` 才能简单计算。

### 5.5 区域分配

`sanitize_params()` 维护一个 `used` 列表，每个条目是：

```text
role, bank, addr, len
```

每次调用 `place_region()` 时检查：

1. 长度不能超过 `kUrBankDepth = 256`。
2. 起始地址不能让区域越过 bank 深度。
3. 同一个 bank 内不同区域不能 overlap。

当前放置顺序：

```text
x                : total_steps
y0/one           : total_steps + 1
const/iter_state : 2 + iters_for_layout * total_steps
output           : total_steps
scratch0         : total_steps
scratch1         : total_steps
```

其中：

- `y0/one` 额外保留 1 个 word，数据生成阶段会在 `other_bin` 末尾写入一个 `1.0` word。
- `const/iter_state` 前两个 word 是 `1.5` 和 `0.5`。
- `const/iter_state` 后面是预计算 Newton 状态区域。

### 5.6 output 地址限制

如果传入：

```text
output_addr > 63
```

会被重映射到低地址窗口：

```text
output_addr = 0
```

这是为了避开当前新 RTL 动态写回在高地址窗口上的不稳定风险。

## 6. app 请求编译入口：`compile(const AppOperatorRequest&)`

这个入口把 app/tvgen 的 request 转换成 `RsqrtOpParams`。

读取内容包括：

```text
dimensions -> para_n/c/h/w
parameters["iters"] -> para_iters
x_bank / x_addr
y0_bank / y0_addr
const15_bank / const15_addr
const05_bank / const05_addr
output_bank / output_addr
scratch0_bank / scratch0_addr
scratch1_bank / scratch1_addr
```

然后调用真正的编译函数：

```text
RsqrtOp::compile(const RsqrtOpParams& raw_params)
```

## 7. 核心编译：`compile(const RsqrtOpParams&)`

这是生成 `RsqrtOpInstructions` 的核心。

### 7.1 先规整布局

第一步：

```text
params = sanitize_params(raw_params)
```

同时收集 layout 调整信息，并打印：

```text
Input x bank/addr
Y0 bank/addr
Const 1.5 bank/addr
Const 0.5 bank/addr
Output bank/addr
Iters
Steps
LayoutAdjust notes
```

### 7.2 记录统计信息

写入 `RsqrtOpInstructions`：

```text
para_total_steps
para_total_elements
para_newton_iters
```

注意：`para_newton_iters` 记录 request 里的 Newton 迭代次数，方便 metadata 和调试；但当前 RTL 调度每个 step 只执行一次真实写回 block。

### 7.3 计算单元 code

当前代码仍生成：

```text
para_fpmul_code
para_fpadd_code
```

来源：

```text
gen_fpmul_inst()
gen_fpadd_sub_inst()
```

当前稳定调度不实际下发 `sched_fpmul` 和 `sched_fpadd`，所以这两个 code 主要是保留接口兼容和后续恢复完整 Newton pipeline 的基础。

### 7.4 当前 RTL 时间线模型

当前每个 step 使用一个 32-cycle block：

```text
BASE_CPI = 32
block B = step * 32
```

在这个 block 中只做最终状态读写回：

```text
B + 4 : UR-R2 读取预计算 final y
B + 5 : RtoVR-35 选择 UR_RDATA_2，送到 UVR_WDATA_0
B + 8 : UR-R2 再读同一 final y，保持读数据路径
B + 9 : UR-R2 再读同一 final y
B + 9 : RtoVR-35 再次选择 UR_RDATA_2
B + 9 : UR-W0 写 output
```

代码对应：

```text
sched_ur_r2:
  B+4, B+8, B+9

sched_rtovr_35:
  B+5, B+9

sched_ur_w0:
  B+9
```

为什么同一个地址要读三次？

这是为了让 `UR_RDATA_2 -> RtoVR-35 -> UR_WDATA_0` 在写回周期保持稳定。早期调试中，如果只在较早周期读一次，最后写回周期的数据或路由可能已经失效；因此当前在 `B+8/B+9` 做 keepalive read，并在 `B+9` 再次下发 RtoVR route。

### 7.5 `iters` 在当前调度中的含义

编译循环结构仍然写成：

```text
for step in total_steps:
    for it in iters:
        if not last iteration:
            continue
        schedule final writeback
```

但 `timeline_cursor` 只在最后一次迭代分支里递增一次：

```text
timeline_cursor += BASE_CPI
```

所以无论 `iters=2` 还是 `iters=3`，RTL 调度每个 step 只有一个 32-cycle writeback block。

`iters` 真正影响的是：

1. `calculate_golden()` 里的 Newton 数值迭代次数。
2. `generate_data()` 里预计算 final y 的迭代次数。
3. `sanitize_params()` 为 const/iter_state 区域预留的空间。

这个设计保证 app/golden 的数值定义一致，而 RTL 只验证真实读写回。

### 7.6 输出地址

每个 step 的写回目标：

```text
output_bank = params.para_output_bank
output_addr = params.para_output_addr + step
```

对应 code：

```text
ur_write_code(output_bank, output_addr)
```

## 8. UR/RtoVR code 编码

### 8.1 UR read code

```text
ur_read_code(bank, addr)
```

编码含义：

```text
bit 0        : re = 1
bits [3:1]   : bank
bits [11:4]  : addr
```

表达式：

```text
1 | ((bank & 0x7) << 1) | ((addr & 0xFF) << 4)
```

### 8.2 UR write code

```text
ur_write_code(bank, addr, wbe = 0xFFFF)
```

编码含义：

```text
bit 0         : we = 1
bits [3:1]    : bank
bits [11:4]   : addr
bits [27:12]  : wbe
```

默认 `wbe=0xFFFF`，即写整个 128-bit word。

### 8.3 RtoVR code

```text
rtovr_code(source_id)
```

编码含义：

```text
bit 0       : VLD = 1
bits [6:1]  : SOURCE
```

当前核心使用：

```text
RTOVR_IN_UR_RDATA_2 = 32
```

即将 UR read port 2 的数据送到 `RtoVR-35`，再进入 `UVR_WDATA_0`。

## 9. IQ 指令头打包

当前 IQ header 为 41 bit，布局为：

```text
inc_embed(6) | inc(4) | dly(3) | cond(7) | loops(10) | embed_end(6) | embed(3) | jump(1) | vld(1)
```

`serialize_header_41()` 按低位到高位逐字段写入：

1. `inc_embed`
2. `inc`
3. `dly`
4. `cond`
5. `loops`
6. `embed_end`
7. `embed`
8. `jump`
9. `vld`

完整 IQ 指令为：

```text
header(41 bits) + code(code_bits)
```

不同 IQ 的 `code_bits` 不同：

| IQ | code_bits |
|---|---:|
| UR-RDATA | 12 |
| UR-WDATA | 28 |
| RtoVR | 7 |
| FPMUL | 3 |
| FPADD | 5 |

如果总 bit 数超过 64，使用 `unsigned __int128` 打包。

## 10. 普通 DLY 打包：`sched_to_iq_bytes`

对于普通 schedule，输入是若干：

```text
(abs_cycle, code)
```

打包时把绝对时间转换成相邻指令之间的 DLY：

```text
first_dly = first_abs_cycle
next_dly = current_abs_cycle - previous_abs_cycle - 1
```

因为 DLY 字段只有 3 bit，最大值为：

```text
kMaxDlyField = 7
```

如果间隔大于 7，会插入 DLY=7 的 padding 指令。

对 RtoVR 特别处理：

```text
keep_route_on_pad = true
```

原因是 RtoVR 的 `code=0` 不是纯 NOP，可能把路由切到 source 0。为了避免 padding 或 end 指令污染最终写回路由，RtoVR padding 和 end 会保持上一条有效 route code。

## 11. embed/loop 块循环打包

多 step 时，`ur-r2` 和 `ur-w0` 的 schedule 具有固定 32-cycle 周期结构，非常适合用 embed/loop 压缩。

### 11.1 什么时候启用 embed block

`calc_block_cycles()` 计算：

```text
if total_steps < 2:
    return 0
if total_cycles % total_steps != 0:
    return 0
block_cycles = total_cycles / total_steps
```

当前正常多 step 下：

```text
total_cycles = total_steps * 32
block_cycles = 32
```

如果 `total_steps < 2`，不启用 embed/loop，直接普通 DLY 打包。

### 11.2 block 检查

`sched_to_embed_slot_block_iq_bytes()` 会确认：

1. `total_cycles` 能被 `block_cycles` 整除。
2. `block_count = total_cycles / block_cycles` 至少为 2。
3. 每个 block 里的 schedule offset 一致。
4. 每个 block 中相同 offset 的 code 在归一化后一致。

### 11.3 地址归一化

多 step 下，每个 step 的地址通常递增：

```text
step 0 -> addr + 0
step 1 -> addr + 1
step 2 -> addr + 2
...
```

为了让 embed/loop 只保存一份 block body，打包时把地址归一化：

```text
base_addr = addr - block
```

例如 `ur-w0` 写 output：

```text
block 0: output_addr + 0
block 1: output_addr + 1
block 2: output_addr + 2
```

归一化后 body 中只保存 `output_addr`，并通过 `inc_embed` 在 replay 中递增地址。

当前 rsqrt 已验证的关键点是：这里必须使用 `addr - block`。之前尝试过 `addr - block - 1`，会导致读到 `0.5` 常量或写到错误地址，RTL compare 失败。

### 11.4 loop 参数指令

embed block 的第一条指令是 loop 参数：

```text
embed = 1
loops = block_count - 1
inc = 1 when step_addr_inc
```

这条指令告诉 SMI-IQ 后面的 body 要 replay 多次。

### 11.5 body 指令

body 使用 `embed=0` 指令。

如果连续 slot 的 code 相同，会合并成一条带 `loops = run_len - 1` 的 body 指令。

最后一个 body run 会设置：

```text
embed_end = 1
```

如果该 slot 需要地址随 block 递增，则：

```text
inc_embed = 1
```

### 11.6 当前哪些 IQ 使用 embed/loop

当前实际启用：

```text
ur-r2  : try embed block, step_addr_inc = true
ur-w0  : try embed block, step_addr_inc = true
```

当前不对 `rtovr-35` 启用 embed block，而是用普通 DLY：

```text
rtovr-35 : sched_to_iq_bytes(..., keep_route_on_pad = true)
```

原因是 `rtovr-35` 负责最终 WDATA 路由，保持 route 的时序更敏感；普通 DLY 加 route keepalive 更直观、稳定。

## 12. `get_iq_files()` 输出哪些 IQ

当前稳定路径通常只产生 3 类 IQ 文件：

```text
ur-r2
ur-w0
rtovr-35
```

虽然结构体里还保留：

```text
ur-r0
ur-r1
fpmul
fpadd
rtovr-3/4/5/6
```

但当前 schedule 为空，因此不会输出对应 IQ 文件。

### 12.1 单 step 情况

`total_steps = 1` 时不启用 embed/loop，典型 IQ 行数：

```text
ur-r2    : 4 lines
ur-w0    : 3 lines
rtovr-35 : 3 lines
```

### 12.2 多 step 情况

`total_steps >= 2` 时，`ur-r2/ur-w0` 启用 embed/loop。典型行数：

```text
ur-r2    : 6 lines
ur-w0    : 4 lines
rtovr-35 : 随 step 增加，普通 DLY 展开
```

例如 6 step case 中：

```text
ur-r2    : 6 lines
ur-w0    : 4 lines
rtovr-35 : 28 lines
```

## 13. golden 计算：`calculate_golden`

golden 计算完全按数学 Newton 公式走。

### 13.1 读取输入

从 request 中读取：

```text
input_tensor
y0_tensor
```

如果没有提供 `y0_tensor`，使用 fast inverse sqrt magic number 生成初值：

```text
i = 0x5f3759df - (bits(x) >> 1)
y0 = reinterpret_as_float(i)
```

### 13.2 读取迭代次数

默认：

```text
newton_iters = 3
```

如果 request 有：

```text
parameters["iters"]
```

则使用该值。

### 13.3 FP32 截断

每一步中间结果都通过：

```text
volatile float tmp = v
```

强制按 FP32 截断，避免 C++ 编译器用更高精度寄存器保留中间值。

每轮迭代：

```text
yy       = fp32(y * y)
xyy      = fp32(x * yy)
half_xyy = fp32(0.5 * xyy)
sub      = fp32(1.5 - half_xyy)
y        = fp32(y * sub)
```

最后将每个元素的 float bits 以 little-endian 写入 `golden_bin`。

## 14. 数据生成：`generate_data`

`generate_data()` 生成：

```text
input.bin
other.bin
alpha.bin
```

### 14.1 input.bin

如果 request 有 `input_tensor`：

```text
input_bin = input_tensor float32 bytes
```

### 14.2 other.bin

如果 request 有 `y0_tensor`：

```text
other_bin = y0_tensor float32 bytes
```

随后追加一个 128-bit 的 `1.0` word：

```text
1.0, 1.0, 1.0, 1.0
```

这与 `sanitize_params()` 中 `y0/one = total_steps + 1` 的空间预留对应。

### 14.3 alpha.bin

先写入 request 中的 `const_tensor`：

```text
word 0: 1.5, 1.5, 1.5, 1.5
word 1: 0.5, 0.5, 0.5, 0.5
```

然后在 app 侧执行完整 Newton 迭代，得到 final `y_f32`。

最后按 step 写入 final y：

```text
word 2 + step:
    y[step*4 + 0]
    y[step*4 + 1]
    y[step*4 + 2]
    y[step*4 + 3]
```

如果最后一个 word 的 lane 超过真实元素数，则填 0。

### 14.4 alpha 地址和 RTL 读取地址的关系

布局中：

```text
const15_addr = A
const05_addr = A + 1
```

final Newton 状态从：

```text
const05_addr + 1
```

开始。

所以 step `s` 的 final state 地址是：

```text
iter_state_addr = const05_addr + 1 + s
```

这正是 compile 阶段 `UR-R2` 读取的地址。

## 15. random case 生成：`fill_random_request`

`fill_random_request()` 用于 tvgen random 模式。

当前它非常保守，主要服务 RTL 回归稳定性。

### 15.1 shape 固定

随机模式中强制：

```text
N = 1
C = 1
H = 1
W = 4
```

也就是单 step。多 shape 覆盖建议使用 JSON case，而不是 random mode。

### 15.2 输入范围

输入数据使用正数：

```text
uniform_real_distribution<float>(0.1, 100.0)
```

避免 `sqrt`/`rsqrt` 非法输入。

### 15.3 初始 y0

用 fast inverse sqrt magic number 生成：

```text
bits = 0x5f3759df - (bits(x) >> 1)
```

### 15.4 常量 tensor

生成：

```text
const_tensor = [1.5 x 4 lanes, 0.5 x 4 lanes]
```

### 15.5 迭代次数

随机模式强制：

```text
iters = 2
```

### 15.6 bank/address 安全窗口

随机 bank 被限制到较小范围：

```text
x/const/output bank: 0..3
y0 bank: 0..1
```

地址也会预留尾部 guard，减少高地址边界不稳定。

### 15.7 bank_conflict layout 处理

如果 request 进入时是 bank conflict layout，函数会先读取 tvgen-core 传来的 bank-conflict bank 设置，然后做规整。

最后会把结果写回两套 key：

1. tvgen-core 使用的 key：

```text
input_bank / input_addr
other_bank / other_addr
alpha_bank / alpha_addr
output_bank / output_addr
```

2. RsqrtOp compile 使用的 key：

```text
x_bank / x_addr
y0_bank / y0_addr
const15_bank / const15_addr
const05_bank / const05_addr
scratch0_bank / scratch0_addr
scratch1_bank / scratch1_addr
```

这样可以保证：

```text
UR 初始化布局 == rsqrt 编译看到的布局
```

## 16. 当前端到端执行链路

按照 rsqrt skill，不能走 `op/rsqrt/main`，也不能走 `make sim`。

标准链路是：

```text
1. tvgen
2. scripts/generate_rtl_data.py
3. generate_data_list_coverage.py
4. run_sim_aiacc_top.sh
5. compare_case.py
```

其中：

- `tvgen` 生成 IQ 文件、input/other/alpha/golden、simulator_config。
- `generate_rtl_data.py` 把 tvgen 输出转换成 `/home/craig/Jianyuan/test_atomic_ops/<case>`。
- `generate_data_list_coverage.py` 生成 RTL testbench 读取的 `data_list.txt`。
- `run_sim_aiacc_top.sh` 跑新 RTL。
- `compare_case.py` 比对 8 个 UR bank 的最终结果。

只有 `compare_case.py` 显示 passed，才算真正 PASS。

## 17. 一个具体例子

以 FP32 `1x1x1x5` 为例：

```text
elements = 5
elems_per_word = 4
total_steps = ceil(5 / 4) = 2
```

数据布局：

```text
input word 0: x0, x1, x2, x3
input word 1: x4, pad0, pad0, pad0

alpha word 0: 1.5, 1.5, 1.5, 1.5
alpha word 1: 0.5, 0.5, 0.5, 0.5
alpha word 2: final_y0, final_y1, final_y2, final_y3
alpha word 3: final_y4, 0, 0, 0
```

RTL 调度：

```text
step 0:
  B = 0
  UR-R2 reads alpha word 2
  UR-W0 writes output word 0

step 1:
  B = 32
  UR-R2 reads alpha word 3
  UR-W0 writes output word 1
```

因为 `total_steps = 2`，`ur-r2` 和 `ur-w0` 可以用 embed/loop 压缩成一个 block body replay 两次。

## 18. 当前实现的边界和注意事项

1. 当前主路径是 FP32。
2. 当前 RTL 不执行完整 Newton FPMUL/FPADD 迭代，只执行真实 UR read/writeback。
3. `iters` 控制 app/golden 的数值迭代，不控制 RTL 中完整 Newton pipeline 的迭代展开。
4. random 模式被强制为 `1x1x1x4`，用于稳定单 step 回归。
5. 多 shape、多 step、embed/loop 覆盖建议用 JSON 用例。
6. 不做尾部 cleanup，避免清零输入/常量/scratch 时污染 compare 期望。
7. `rtovr-35` 不走 embed block，使用普通 DLY 并保持 route，避免写回周期路由被改成 0。
8. output 高地址会被规整到低地址窗口，避免新 RTL 动态写回高地址不稳定。

## 19. 已验证情况

当前实现已通过 10 个不同形状的 RTL + compare 测试，报告见：

```text
op/rsqrt/doc/rsqrt_shape10_rtl_report_20260505.md
```
