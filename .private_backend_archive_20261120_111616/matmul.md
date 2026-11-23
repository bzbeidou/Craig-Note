# 矩阵乘：语义、布局与性能边界

## 1. 运算语义

对矩阵

$$
A\in\mathbb{R}^{M\times K},\qquad B\in\mathbb{R}^{K\times N},
$$

矩阵乘定义为

$$
C_{m,n}=\sum_{k=0}^{K-1}A_{m,k}B_{k,n},\qquad C\in\mathbb{R}^{M\times N}.
$$

$M$ 和 $N$ 构成输出域，$K$ 是规约维。若把一次乘法和一次加法分别计作一个浮点运算，则经典 GEMM 的运算量近似为 $2MNK$ FLOPs；边界项的精确计数为每个输出 $K$ 次乘法和 $K-1$ 次加法。

编译器处理中应同时记录：逻辑形状、数据类型、累加类型、转置属性、批维、布局和步长。仅有 $M/N/K$ 不足以确定合法且高效的代码。

## 2. MatMul、GEMM 与 GEMV

| 名称 | 典型形式 | 后端关注点 |
|---|---|---|
| MatMul | $C=AB$ | 通用矩阵乘语义，可能包含批维和动态形状 |
| GEMM | $C=\alpha AB+\beta C$ | 融合缩放、累加以及 epilogue |
| GEMV | $y=Ax$ | 一维输出较窄，通常受带宽和规约延迟限制 |

大语言模型的 prefill 往往形成较大的矩阵乘，较容易提高矩阵单元利用率；decode 中的 token 批量较小，常呈现 GEMV 或 skinny GEMM 特征。这里的结论不是由算子名称决定，而应由实际形状、批处理策略、KV cache 访问和量化格式共同判断。

## 3. 从数学表达式到硬件指令

矩阵引擎执行的通常是固定或有限集合的微块：

$$
A_{tile}[M_t,K_t]\times B_{tile}[K_t,N_t]\rightarrow C_{tile}[M_t,N_t].
$$

后端需要完成四类映射：

1. **迭代空间分块**：选择 $M_t/N_t/K_t$ 以及外层遍历次序；
2. **存储分层**：确定 DRAM、片上 SRAM、寄存器或专用累加器中的驻留范围；
3. **布局映射**：把逻辑索引映射为 lane、bank 和物理地址；
4. **流水调度**：重叠搬运、矩阵计算、逐元素 epilogue 和写回。

这也是现代 tile DSL 的核心抽象。TileLang 将数据流与线程绑定、布局、tensorization 和 pipeline 等调度决策分离；ThunderKittens 则以寄存器 tile、共享存储 tile 和异步 worker 模板表达同类问题。两者不能直接移植到 BM1684X，但其 IR 设计思想适合用于重构本地后端。

## 4. 布局、步长与转置

行主序、列主序只是二维矩阵的两种常见线性化方式。AI 加速器还会使用 blocked、interleaved、lane-distributed 或 swizzled 布局。布局决定：

- 连续访存方向及突发传输效率；
- 同一 tile 是否分布到正确的 lane；
- 访问是否产生 bank conflict；
- 数据能否直接满足矩阵指令的输入格式；
- 是否需要显式 transpose、pack 或 layout conversion。

GEMM 接口中的 `NN/NT/TN/TT` 描述两个输入是否按转置语义读取。转置属性不必然触发完整的数据搬移；编译器可通过交换步长、改变 tile 读取方式或选择不同指令变体实现逻辑转置。

## 5. 性能模型

单纯使用峰值 FLOPs 无法解释实际性能。至少应同时评估：

$$
T\ge \max(T_{compute},T_{memory},T_{dependency})+T_{overhead}.
$$

其中 $T_{compute}$ 受 tile 填充率和指令吞吐约束，$T_{memory}$ 受数据复用和带宽约束，$T_{dependency}$ 描述同步与流水气泡，$T_{overhead}$ 包括启动、布局转换和尾块处理。Roofline 模型可作为第一层判断，但对于多引擎异步执行，还需要资源占用和事件依赖模型。

建议记录以下指标：有效 FLOPs、实际传输字节数、算术强度、矩阵单元利用率、片上存储占用、尾块浪费、搬运与计算重叠率，以及端到端延迟。

## 6. 前沿方向

- **低比特与块级缩放**：把存储类型、计算类型、累加类型和缩放元数据分别建模。Ladder 的 tType 表明，低比特优化不能简化为替换一个 `dtype`。
- **动态形状的多版本生成**：针对 prefill、decode、batch 波动生成有限组 shape-specialized kernel，并由运行时选择。
- **融合与持久化执行**：将 GEMM epilogue、归一化、激活或通信纳入 tile 级任务图，减少中间结果回写。
- **可验证变换**：为布局转换、重关联和低精度改写定义可检查的等价条件，避免搜索系统只优化性能而破坏数值语义。

## 参考

- [TileLang: A Composable Tiled Programming Model for AI Systems, 2025](https://arxiv.org/abs/2504.17577)
- [ThunderKittens, ICLR 2025](https://proceedings.iclr.cc/paper_files/paper/2025/hash/05dc08730e32441edff52b0fa6caab5f-Abstract-Conference.html)
- [Ladder, OSDI 2024](https://www.usenix.org/conference/osdi24/presentation/wang-lei)

