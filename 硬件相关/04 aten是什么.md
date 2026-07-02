### 1. `aten` 的本源含义

`aten` 来自 **PyTorch ATen（A Tensor Library）**：

- PyTorch 的 **算子抽象层**
- `aten::add`
- `aten::rsqrt`
- `aten::matmul`

在 PyTorch IR / ONNX / FX 里：

- `aten` 表示 **“框架语义级算子”**
- 还没绑定到具体硬件

### 2. 在你这个工程里的真实含义（非常重要）

在 **SMI-IQ / aiacc** 这个项目中：

> **`emit_aten_xxx` 的意思是：**
>
> “将 *框架语义层* 的算子，Lower 成 *硬件可理解的微指令序列*”