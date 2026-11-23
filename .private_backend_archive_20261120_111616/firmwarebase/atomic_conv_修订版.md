# `atomic_conv_gen_cmd`：从卷积语义到硬件描述符

## 1. 文档范围

`atomic_conv_gen_cmd` 的职责不是执行卷积，而是把卷积参数、LMEM 地址、布局和依赖信息编码为 BD 引擎描述符。分析该函数时应区分四个阶段：

1. 规范化几何参数；
2. 规范化布局与 stride；
3. 检查 ISA 约束；
4. 生成命令，并在可选构建中记录分析型性能画像。

本文件中的字段与约束属于源码事实；对内部数据流的解释属于架构分析；周期公式属于分析模型，不等同于真机时延。

## 2. 卷积几何

有效卷积核尺寸为

$$
K_h^{eff}=d_h(K_h-1)+1,\qquad K_w^{eff}=d_w(K_w-1)+1.
$$

考虑输入插洞与四边 padding 后，有效输入尺寸为

$$
I_h^{eff}=(I_h-1)(ins_h+1)+p_t+p_b+1,
$$

$$
I_w^{eff}=(I_w-1)(ins_w+1)+p_l+p_r+1.
$$

输出尺寸为

$$
O_h=\left\lfloor\frac{I_h^{eff}-K_h^{eff}}{s_h}\right\rfloor+1,
\qquad
O_w=\left\lfloor\frac{I_w^{eff}-K_w^{eff}}{s_w}\right\rfloor+1.
$$

这里的 $s_h/s_w$ 是卷积几何步长；`input_stride[4]` 是内存布局步长。两者必须在 IR 和 API 中使用不同类型或命名，避免将几何属性误传为地址属性。

## 3. 输入布局规范化

调用方未传入 `input_stride` 时，生成器选择默认 aligned 布局；传入时选择 user-stride 模式。显式 stride 的顺序为 N/C/H/W，单位为元素。

两项规范化规则为：

```c
str[0] = input_n == 1 ? 0 : str[0];
str[1] = (input_c + input_addr / LOCAL_MEM_SIZE) <= NPU_NUM
       ? 0 : str[1];
```

第一项成立是因为单 batch 不会访问下一 N；第二项成立是因为当全部通道分别落在不同 lane 时，同一 lane 内不存在第二个通道，C 方向偏移不会被使用。这是字段正规化，不是计算语义变换。

通道分布可表示为：

$$
lane(c)=(lane_0+c)\bmod64,
$$

$$
offset(c)=offset_0+\left\lfloor\frac{lane_0+c}{64}\right\rfloor C_{stride}+\cdots.
$$

## 4. 合法性约束

### 4.1 地址归属

| 约束 | 语义 |
|---|---|
| 权重与输出具有相同起始 lane | 权重按输出通道组织 |
| bias 与输出具有相同起始 lane | bias 按输出通道组织 |
| pad/insert 表与输入具有相同起始 lane | 表项按输入通道组织 |
| 非 FP32 输入从规定的 lane 起点开始 | 来自当前生成器断言；微架构原因需单独验证 |

常数操作数不占用相应 LMEM 地址，因此豁免地址归属检查。

### 4.2 对齐

| 对象 | 约束摘要 |
|---|---|
| 默认布局输入 | 64 字节对齐 |
| user-stride 输入 | 至少按元素大小对齐 |
| 非 FP32 权重 | 64 字节对齐 |
| bias | 4 字节对齐 |
| pad/insert 表 | 两个元素的对齐粒度 |
| 输出 | FP32 为 64 字节；其他当前路径为 8 字节 |

对齐事实可由断言验证；“为什么是该粒度”的解释若没有硬件文档，只能标注为推断。

### 4.3 精度组合

当前浮点路径允许：FP16→FP16/FP32、BF16→BF16/FP32、FP32→FP32。后端应把该关系编码为目标合法化规则，而不是依赖 cmodel 中的末端断言。

### 4.4 位域范围

N/C/H/W、输出形状、核形状和有效输入尺寸受 16 位字段约束；stride、padding、dilation 和 insertion 受 4 位字段约束。推导量 `ih_ext/iw_ext` 必须独立检查，因为合法输入参数的乘积仍可能溢出字段。

这些断言被 `USING_CMODEL` 条件包围时，意味着仿真构建能主动报告问题，而设备构建未必具备同等防护。编译器应在生成描述符前重复关键检查，不能把 cmodel 当作唯一防线。

## 5. 命令生成

描述符至少包含：

- `CMD_ID` 与 `CMD_ID_DEP`；
- CONV 任务类型和模式标志；
- 输入/输出精度及常数标志；
- stride、padding、dilation、insertion；
- 输入、权重、bias、输出和 pad/insert 的地址；
- 输入/输出/卷积核形状；
- user-stride 字段。

应为描述符建立机器可读 schema，并自动生成：字段范围检查、编码器、解码器、短长格式转换以及 round-trip 测试。这样可减少手写位移与文档表格长期漂移。

## 6. 性能画像

`CONV_GET_PROFILE` 仅在相应统计构建开关启用时调用 `atomic_conv_get_profile`，结果写入 `pid_node->inst_profile`。它不改变卷积数值语义，也不应被理解为硬件计时。

当前分析模型可写为

$$
T_{conv}=N
\left\lceil\frac{OC}{64}\right\rceil
\left\lceil\frac{OH\cdot OW}{P}\right\rceil
\left\lceil\frac{IC}{G}\right\rceil
K_hK_w,
$$

其中 $G$ 为输入通道并行度，$P$ 为输出位置并行度。对当前模型：INT8 的 $G=64,P=4$；FP16/BF16 的 $G=32,P=4$；FP32 需要按源码中的精度分支单独确认。

运算量按一次 MAC 计两个 operation：

$$
Ops=2N\cdot OC\cdot OH\cdot OW\cdot IC\cdot K_h\cdot K_w.
$$

可定义理想计算利用率：

$$
\eta_{compute}=\frac{Ops}{T_{conv}\cdot Ops_{peak/cycle}}.
$$

该指标反映 tile 填充，不包含 GDMA、bank conflict、命令启动、跨引擎等待和频率变化。

## 7. 从分析模型升级到校准模型

推荐采用混合模型：

$$
T_{measured}=T_{analytic}
+T_{startup}+T_{memory}+T_{contention}+T_{dependency}+\epsilon.
$$

实施步骤：

1. 用合成形状覆盖 OC/IC/P 的整除与尾块情况；
2. 单独测量命令启动和纯 GDMA；
3. 改变 LMEM bank 与并发引擎，估计争用项；
4. 以解析模型为先验拟合残差；
5. 用未见 shape、动态 shape 和真实网络验证排序能力。

用于调度搜索时，候选排序相关性通常比单点周期误差更重要，二者都应报告。

## 8. 可研究方向

- **约束上移**：从 atomic 断言自动生成 MLIR/后端 verifier；
- **描述符超优化**：联合搜索短/长格式、tile、布局和流水，并以等价性检查过滤候选；
- **依赖最小化**：基于 LMEM 区间与数据流推导最小 `CMD_ID_DEP`，消除冗余等待；
- **在线校准**：让 profiler 将测量结果回写到按芯片版本区分的代价模型；
- **数值契约**：把累加精度、舍入、饱和和 `result_add` 语义纳入验证。

相关研究可参考 [PipeThreader](https://www.usenix.org/conference/osdi25/presentation/cheng)、[KPerfIR（OSDI 2025 论文列表）](https://www.usenix.org/conference/osdi25/technical-sessions) 与 [Mirage](https://www.usenix.org/conference/osdi25/presentation/wu-mengdi)。

