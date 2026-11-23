# 矩阵乘的

## 1. “M–N–K ”的准确含义

矩阵乘

$$
C_{m,n}=\sum_k A_{m,k}B_{k,n}
$$

可以表示为三维迭代域

$$
\mathcal{D}=\{(m,n,k)\mid 0\le m<M,\ 0\le n<N,\ 0\le k<K\}.
$$

几何上的长方体表示计算实例集合，不表示某个三维张量。$(m,n)$ 标识输出元素，$k$ 标识该输出的规约项。将该迭代域投影到 $M\times N$ 平面得到输出域，沿 $K$ 轴归约得到最终结果。

## 2. 分块是仿射变换

设 tile 尺寸为 $(M_t,N_t,K_t)$，则可将索引写为

$$
m=m_oM_t+m_i,\quad n=n_oN_t+n_i,\quad k=k_oK_t+k_i.
$$

这一变换把原始三层循环分为 tile 间和 tile 内两个层级。编译器还可以进行循环交换、融合、展开、向量化和 tensorization。合法性取决于依赖关系；性能取决于复用、并行度、片上容量和硬件指令形状。

对一个输出 tile：

$$
C_{tile}^{(k_o+1)}=C_{tile}^{(k_o)}+A_{tile}^{(k_o)}B_{tile}^{(k_o)}.
$$

因此 $K$ 分块需要跨 tile 保存部分和。部分和驻留在累加器、LMEM 还是外部内存，会直接决定带宽和数值行为。

## 3. 尾块与有效利用率

当 $M/N/K$ 不能整除硬件 tile 时，需要 padding、predicate、mask 或专用尾块指令。仅从计算阵列填充率看，可定义

$$
\eta_{tile}=\frac{MNK}
{\lceil M/M_t\rceil M_t\cdot
 \lceil N/N_t\rceil N_t\cdot
 \lceil K/K_t\rceil K_t}.
$$

$\eta_{tile}$ 只描述形状浪费，不包含访存、依赖、流水和频率等因素，不能直接当作端到端利用率。

## 4. 指令命名的解释

类似 `m16n16k16` 的名称通常描述一次矩阵指令覆盖的迭代域：输出 tile 为 $16\times16$，规约深度为 16。它并不产生一个 $16\times16\times16$ 的三维结果。不同厂商对操作数布局、输入精度、累加精度和线程所有权的定义不同，不能仅凭名称推断二进制语义。

目标加速器的矩阵乘分组可用相同方法理解：$M$ 方向映射到 lane，$N$ 方向对应一组输出位置，$K$ 方向由加法树规约。具体并行宽度记为规约宽度 $G(dtype)$ 与输出位置宽度 $P(dtype)$，其数值在公开版本中不展开。

## 5. 从可视化到编译器 IR

三维迭代域适合成为后端 IR 的第一层表示，但仍需补充：

- `iterator_types`：parallel 或 reduction；
- `indexing_maps`：各操作数如何由 $(m,n,k)$ 寻址；
- `layout_map`：逻辑元素到 lane/bank/address 的映射；
- `memory_space`：外存、LMEM、寄存器或累加器；
- `async_dependency`：tile 搬运、计算和写回之间的事件；
- `numeric_contract`：舍入、溢出、累加精度和可重关联条件。

这与 MLIR 的结构化算子、Transform Dialect，以及 Mirage 的多层级图思想相呼应。对本后端而言，值得探索的不是复制某套 GPU IR，而是建立能同时表达“迭代域—布局—指令字段—异步依赖”的最小 IR。

## 参考

- [Towards a high-performance AI compiler with upstream MLIR, 2024](https://arxiv.org/abs/2404.15204)
- [Mirage: A Multi-Level Superoptimizer for Tensor Programs, OSDI 2025](https://www.usenix.org/conference/osdi25/presentation/wu-mengdi)

