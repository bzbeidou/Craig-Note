# LMEM、Lane、EU 与 Bank

## 1. 存储层次中的位置

BM1684X 的 LMEM 是靠近张量计算单元、由软件显式管理的片上 SRAM。典型数据路径为：

```text
系统内存 --GDMA/S2L--> LMEM --BD/TIU--> LMEM --GDMA/L2S--> 系统内存
```

LMEM 不是透明缓存。编译器或固件负责分配地址、选择布局、建立搬运与计算之间的依赖，并保证生命周期不重叠。

## 2. 组织结构

根据当前 BM1684X 软件模型：

| 层级 | 数量/容量 | 作用 |
|---|---:|---|
| Lane/NPU | 64 | 计算并行与通道分布 |
| 每 lane 的 LMEM | 256 KiB | 保存该 lane 所属数据 |
| 每 lane 的 bank 区域 | 16 × 16 KiB | 提供可并行访问的 SRAM 组织 |
| TPU Core 的总 LMEM | 16 MiB | 64 × 256 KiB |

统一地址可按当前模型解释为：

```text
| lane [23:18] | bank [17:14] | bank 内偏移 [13:0] |
```

即

$$
lane=addr\gg18,\quad bank=(addr\gg14)\mathbin{\&}0xf.
$$

这是软件地址模型。若要推断端口数量、仲裁方式或物理布线，仍需硬件规格或微基准验证。

## 3. Lane、EU 与 Bank 的职责

| 概念 | 回答的问题 |
|---|---|
| Lane/NPU | 哪一组计算通路和本地存储拥有该逻辑通道？ |
| EU | 一个 lane 内一次可以处理多少元素？ |
| Bank | 同一时间的多个存储请求能否由片上 SRAM 并行服务？ |

当前 aligned layout 以 64 字节为基本向量宽度：FP32/INT32 对应 16 个元素，FP16/BF16/INT16 对应 32 个元素，INT8 对应 64 个元素。对 $H\times W=20$ 的单通道数据，aligned stride 将向上对齐到相应元素数；这会增加容量占用，但通常简化向量取数。

通道按 lane 轮转分布可写为：

$$
lane(c)=(lane_0+c)\bmod64,
$$

同一 lane 中的通道组号为

$$
group(c)=\left\lfloor\frac{lane_0+c}{64}\right\rfloor.
$$

该映射应当进入布局分析，而不应散落为多个算子中的隐式约定。

## 4. Bank conflict 的判定

Bank conflict 是资源冲突，不是地址别名。若一组时间上重叠的访问映射到同一 bank，且请求数超过该 bank 在该时段可提供的端口或服务能力，则访问会被串行化或产生停顿。

形式化地，对同一调度时隙中的访问集合 $A$，可定义某 bank 的请求负载：

$$
L_b=\sum_{a\in A}\mathbf{1}[bank(a)=b]\cdot cost(a).
$$

当 $L_b$ 超出可用服务能力时存在冲突风险。`cost(a)` 需要区分读写类型、向量宽度、广播、GDMA/TIU 竞争等因素，因此仅比较起始地址所在 bank 并不充分。

地址覆盖则是活跃区间在字节范围上相交，并且至少存在一次破坏性写入。前者主要影响性能，后者可能破坏正确性。

## 5. 编译器中的 LMEM 规划

一个完整的 LMEM 分配器至少需要处理：

1. **容量约束**：每 lane 的局部占用不能越界；
2. **对齐约束**：满足指令和 DMA 的地址粒度；
3. **归属约束**：输入、权重、bias、输出落到指令要求的 lane；
4. **生命周期约束**：互相干涉的张量不能复用同一地址区间；
5. **bank 代价**：对同时访问的缓冲区选择合适的 bank 偏移；
6. **流水约束**：双缓冲会扩大容量占用，但能够隐藏搬运延迟。

这可建模为带有硬约束和软代价的分配问题：越界、别名和归属属于硬约束；bank conflict、碎片和 DMA 对齐损失属于优化目标。

## 6. 建议的验证微基准

- 固定计算形状，仅改变两个输入的 bank 间距，测量周期变化；
- 固定 bank，改变读/写组合，推断端口与仲裁特征；
- 让 GDMA 与 BD/TIU 并发访问相同或不同 bank，测量干扰；
- 对 aligned/compact/custom stride 分别测量带宽和容量开销；
- 将分析结果与 cmodel、真机计数器交叉验证。

前沿 GPU DSL 已把 layout 和 swizzle 提升为一等调度对象。对 BM1684X，更有价值的方向是建立 lane/bank-aware layout IR，使 bank 代价能参与 layer-group 和 tiling 搜索，而不是在地址分配完成后被动诊断。

## 参考

- [ThunderKittens, ICLR 2025](https://proceedings.iclr.cc/paper_files/paper/2025/hash/05dc08730e32441edff52b0fa6caab5f-Abstract-Conference.html)
- [Ladder, OSDI 2024](https://www.usenix.org/conference/osdi24/presentation/wang-lei)

