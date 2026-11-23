# BM1684X 后端与 AI 编译器学习笔记

本目录以 BM1684X 后端为主要案例，讨论张量计算、片上存储、指令调度、性能建模与验证方法。文档不把特定芯片实现直接推广为行业共性，而是区分四类结论：

| 标记 | 含义 | 使用方式 |
|---|---|---|
| **源码事实** | 可由当前仓库的宏、断言、字段或参考模型直接验证 | 给出符号名或源码位置 |
| **架构解释** | 对源码事实的机制化解释 | 必须说明推理链，不视为官方规格 |
| **分析模型** | 用于估算周期、带宽或利用率的简化模型 | 不等同于真机测量 |
| **研究假设** | 尚待实验验证的编译器方案 | 给出变量、基线和评价指标 |

## 阅读顺序

1. [矩阵乘：语义、布局与性能边界](matmul.md)
2. [矩阵乘的迭代空间与分块](mm的立体视角展示.md)
3. [矩阵指令的粒度：软件分块与硬件遍历](最小块的执行方式.md)
4. [BM1684X 的 Cube 数据通路](bm里的cubu概念.md)
5. [LMEM、Lane、EU 与 Bank](lmem在tpu上的分布.md)
6. [静态调度、动态调度与异构流水](静态调度vs动态调度.md)
7. [后端测试分层与验证策略](tpuDNN和test的区别.md)
8. [CONV 指令生成与性能模型](firmwarebase/atomic_conv_修订版.md)
9. [BM1684X CONV 指令参考](firmwarebase/BM1684X_CONV指令参考.md)
10. [可研究的编译器方向](编译器研究方向.md)

## 当前研究脉络

截至 2026 年，AI 编译器的主要矛盾已从“能否生成张量指令”转向“能否联合表达并优化数据移动、异构流水、布局、精度与跨算子执行”。代表性工作包括：

- [TileLang（2025）](https://arxiv.org/abs/2504.17577)：将数据流与线程绑定、布局、tensorization、流水等调度决策分离；
- [PipeThreader（OSDI 2025）](https://www.usenix.org/conference/osdi25/presentation/cheng)：以 tile 级任务图搜索计算与专用搬运单元之间的软件流水；
- [Mirage（OSDI 2025）](https://www.usenix.org/conference/osdi25/presentation/wu-mengdi)：联合搜索代数变换与多层级调度，并加入等价性验证；
- [Twill（OSDI 2026）](https://www.usenix.org/conference/osdi26/presentation/soi)：把软件流水与 warp specialization 表述为约束优化问题；
- [MPK（OSDI 2026）](https://www.usenix.org/conference/osdi26/presentation/cheng)：将跨算子任务下沉到持久化 mega-kernel，以细粒度任务图组织计算和通信；
- [Ladder（OSDI 2024）](https://www.usenix.org/conference/osdi24/presentation/wang-lei)：把低比特存储、类型转换和硬件支持的计算类型纳入统一优化；
- [TLM（OSDI 2024）](https://www.usenix.org/conference/osdi24/presentation/zhai)：使用学习模型引导张量程序搜索，但仍以编译、运行和测量形成闭环。

这些工作为本目录提供了一个统一视角：后端优化的对象不是孤立指令，而是带有依赖、布局、资源占用和数值语义的 tile 级程序。

