#### 1、通过ONNX Operators列表了解常见算子

#### 2、超出2GB的错误

ONNX模型文件大小超过了Protocol Buffers（protobuf）的2GB限制导致的。LLama2-7B作为大型模型，其ONNX格式在序列化时可能包含海量参数和计算节点，直接调用`check_model`会导致内存溢出。

#### 3、导出为onnx格式模型gpu显存不足   导出失败 

3060只有6GB的显存，导出失败

解决方案：

​	使用CPU导出

  或者降低输入长度

会

#### 4、降低输入长度   --影响导出的 ONNX 模型的输入结构

**例如：**

```python
inputs = tokenizer(text, return_tensors="pt", padding=True, max_length=32, truncation=True)

```

**也就是说只能在已经限定的维度下设置输入长度**

**静态维度**和**动态维度**



上面没有设置动态维度，只能限制长度



动态维度下支持任意输出长度，通用性更强

这里的设置了dynamic_axes这里输出就是动态维度

```
torch.onnx.export(
    model,
    args=(inputs["input_ids"],),
    f=onnx_output_path,
    input_names=["input_ids"],
    output_names=["logits"],
    dynamic_axes={
        "input_ids": {0: "batch", 1: "seq_len"},
        "logits": {0: "batch", 1: "seq_len"},
    },
    opset_version=17,
)

```



#### 5、动态维度和静态维度

深度学习模型中，输入输出张量的形状通常包括以下几个维度

`batch_size`（批大小）：一次送入模型处理的数据样本数量。

`seq_len`（序列长度）：例如文本的长度（token 数）。

`channels / height / width`：图像的空间维度等。

**静态维度：**所有输入输出张量的维度大小都是固定的

```
input_ids: [1, 128] → 表示 batch_size=1, seq_len=128
```

**ONNX 导出时使用静态维度**，意味着模型只能接受这种精确形状的输入。

缺点是：

- 不支持任意长度输入，**无法动态处理更长或更短的文本**
- 必须重新导出 ONNX 才能支持不同输入形状



**动态维度：**在导出模型时，指定某些维度（例如 batch_size、seq_len）为动态的 —— 即可以是任意值。

```
dynamic_axes = {
    "input_ids": {0: "batch", 1: "seq_len"},
    "logits": {0: "batch", 1: "seq_len"}
}
```

通用性强

**为什么原始模型不需要设置这个，但是onnx格式需要？**

  PyTorch 是动态计算图框架，在 PyTorch 中，模型每次前向传播时都会构建一张新的计算图，自动适配不同的输入 shape。

```python
model(tokenizer("hello", return_tensors="pt"))
model(tokenizer("hello world! This is llama2", return_tensors="pt"))
# 这些输入长度不同，PyTorch 会自动处理。
```

**为什么onnx需要设置这个**

  onnx时静态图模型，ONNX 是一个中间表示（IR），用于跨平台、高性能的模型推理。计算图是**提前编译好的静态图**，要求输入输出的 shape 明确或标明哪些维度是动态的。如果你不指定 `dynamic_axes`，ONNX 默认会把你传入的张量 shape 当作 **唯一合法形状**。

```
input_ids.shape == [1, 128]

```

如果没有设置 `dynamic_axes`：

- 导出的 ONNX 模型只接受 `[1, 128]` 形状的输入

如果设置了 `dynamic_axes={"input_ids": {0: "batch", 1: "seq_len"}}`：

- 导出的 ONNX 模型接受任意 `batch_size` 和 `seq_len`

#### 6、 输入设置input_ids以及attention_mask的用处

**(1)input_ids**    是token在词表中的编号

比如句子 `"Hello world"`，通过分词器（Tokenizer）变成 token：`["Hello", "world"]`，再映射为编号 `[15496, 995]`，就是 `input_ids`。

**(2)attrntion_mask**

训练或推理中，我们经常会把多个不同长度的文本 **padding（补零）** 到一样长

| 真实输入句子    | input_ids              | attention_mask |
| --------------- | ---------------------- | -------------- |
| `"Hello world"` | `[15496, 995, 0, 0]`   | `[1, 1, 0, 0]` |
| `"I am a bot"`  | `[40, 716, 257, 2057]` | `[1, 1, 1, 1]` |

`attention_mask=1` 表示模型应“关注”的 token。

`attention_mask=0` 表示这是 padding，模型应该忽略这个位置。



**导出为onnx时也可以设置动态维度**

```
torch.onnx.export(
    model,
    args=(input_ids, attention_mask),
    f="model.onnx",
    input_names=["input_ids", "attention_mask"],
    output_names=["logits"],
    dynamic_axes={
        "input_ids": {0: "batch", 1: "seq_len"},
        "attention_mask": {0: "batch", 1: "seq_len"},
        "logits": {0: "batch", 1: "seq_len"}
    },
    opset_version=17
)

```

导出的模型会**支持任意 batch size 和任意长度的输入序列**