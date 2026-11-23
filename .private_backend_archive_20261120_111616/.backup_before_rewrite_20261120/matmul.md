数学上的 MatMul → 算子里的 GEMM → 硬件里的矩阵计算单元    这几个点要区分开

看到mm第一反应不要只是：“这是两个矩阵相乘。”应该马上把它翻译成：

$$ \boxed{[M,K]\times[K,N]\rightarrow[M,N]} $$

```
M 多大？N 多大？K 多大？
dtype 是什么？
一次硬件算多大的 tile？
```

## 一、基本的矩阵乘

假设：$$ A\in \mathbb{R}^{M\times K} $$ $$ B\in \mathbb{R}^{K\times N} $$

那么：$$ C=A\times B $$

结果：$$ C\in \mathbb{R}^{M\times N} $$

**矩阵乘最重要的三个字母**：

**$$ \boxed{M,\ N,\ K} $$**一定要非常熟。

例如：

$$ A= \begin{bmatrix} 1&2&3\\ 4&5&6 \end{bmatrix} $$

A 的 shape：$$ 2\times3 $$

所以：$$ M=2,\quad K=3 $$

如果：

$$ B= \begin{bmatrix} 1&2\\ 3&4\\ 5&6 \end{bmatrix} $$

shape 是：$$ 3\times2 $$

于是：$$ N=2 $$

最终：$$ C=A B $$

shape 是：$$ 2\times2 $$

具体：

$$ C_{0,0} = 1\times1+ 2\times3+ 3\times5 =22 $$

矩阵乘就是批量的A 的一行和 B 的一列做点积。

## 二、中间那个 K 特别重要

矩阵乘：$$ [M,K]\times[K,N]\rightarrow[M,N] $$

计算一个输出：$$ C_{ij} $$

实际上是：$$ C_{ij} = \sum_{k=0}^{K-1} A_{ik}B_{kj} $$

也就是说：$$ C_{ij} = A_{i0}B_{0j} + A_{i1}B_{1j} +\cdots+ A_{i,K-1}B_{K-1,j} $$

这里有：K 次乘法K-1 次加法

因此 K 也经常叫：**Reduction Dimension（规约维度）**

矩阵乘本质上其实就是：乘法 + **沿 K 维度求和**

即：$$ C_{ij}=\text{ReduceSum}_K(A_{ik}B_{kj}) $$这句话对于编译器特别重要。

矩阵乘最底层其实就是 MAC

硬件里有一个非常重要的概念：$$ a\times b+c $$

叫：MAC：Multiply-Accumulate乘累加。

矩阵乘：$$ C_{ij} = \sum_k A_{ik}B_{kj} $$

其实就是不停执行：$$ C_{ij} \leftarrow C_{ij} + A_{ik}B_{kj} $$

**而 Tensor Core / Cube Unit，本质上就是：一次并行执行大量 MAC 的专用硬件。NVIDIA, 升腾**

 GPU / AI 芯片做了一件事情：做一个专门负责“小矩阵 × 小矩阵”的硬件。

比如概念上：

```
      B tile
       ↓
┌────────────────┐
│ Matrix Engine  │
│                │
│ 大量乘法器      │
│ 大量加法器      │
│ 并行 MAC       │
└────────────────┘
       ↓
     C tile
```

NVIDIA 叫：Tensor Core

昇腾 AI Core 中对应矩阵计算能力通常会看到：Cube Unit

虽然具体架构不同，现在可以先统一理解成：**Matrix Engine / Matrix Multiply Unit**

## 三、GEMM、GEMV、MatMul 分别是什么

MatMul泛指矩阵乘：

$$ C=A B $$

GEMM  General Matrix Multiply：

$$ C=\alpha AB+\beta C $$

理解成大规模普通矩阵乘。AI 芯片最重要的算子之一就是 GEMM。

GEMV  Matrix × Vector：

$$ y=A x $$

例如$$ [4096,4096] \times [4096,1] $$

这里：$$ N=1 $$

就是 GEMV。这在 LLM Decode 阶段特别值得关注。因为：Prefilltoken 很多，M 较大，是 GEMM

Decode一次通常只有少量新 token， M 很小，更像 GEMV / skinny GEMM

这就是为什么：LLM Prefill 和 Decode 的硬件利用率差异很大。

## 四、另一个非常重要的概念：数据布局

数学上：

$$ A= \begin{bmatrix} 1&2&3\\ 4&5&6 \end{bmatrix} $$

但是内存实际上是一维的。

一种存法：1 2 3 4 5 6叫：Row Major按行存。

另一种：1 4 2 5 3 6 叫：Column Major按列存。

这会极大影响：Matrix Engine 怎么加载数据。

因此 Tensor Core / Cube Unit 资料里会经常看到：layout，stride，transpose，row-major，column-major，ND，NZ，fragment layout

这些看起来不像数学问题，但其实是：如何把矩阵喂给矩阵乘单元。

## 五、转置表示 NN NT

所以 GEMM API 经常会写：

```
NN
NT
TN
TT
```

例如：

```
C = A × B
C = A × B^T
C = A^T × B
```

N：

```
No Transpose
```

T：

```
Transpose
```