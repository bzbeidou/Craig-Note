#### 1、输入记录

**（1）attention 的 RMSNorm算子结构应该是什么样的呢，输入形状是什么呢**

```
Input (x)
  │
  ├──→ Pow（x^2）
  │
  ├──→ ReduceMean (axis=-1)  → shape: [B, S, 1]
  │
  ├──→ Add（加 epsilon）
  │
  ├──→ Sqrt
  │
  ├──→ Div（x / sqrt(...)）
  │
  └──→ Mul（乘权重 w，通常是 Constant 或 Parameter）
       ↓
     输出

```

输入是经过embedding层的数据，形状为 `[batch, seq_len, hidden_dim]`



**（2）attention层的输入（不包括RMSNorm的部分）**





#### 2、输出unknown的原因其二

Reshape节点为例

```
🔹输入:
  - /model/model/layers.0/self_attn/k_proj/MatMul_output_0: shape=(1, 128, 4096)
  - /model/model/layers.0/self_attn/Concat_1_output_0: shape=(4,), dtype=int64

```

虽然两个输入形状都有，但是Reshape需要需要Concat内的值，并不是类似MatMul这种有形状就能推理的

**实际上在Concat节点就没有值**

在Concat的输入一般如下

```
[
  Unsqueeze(32),        # 动态计算，不是 initializer
  Unsqueeze(128),       # 同上
  Constant(1),          # 是 initializer
  Constant(128)         # 是 initializer
]

```

这些 `Unsqueeze(x)` 的值本身无法在图里找到 initializer，因此整个 `Concat` 的输出也没有值，你无法解析出 reshape 的目标。


