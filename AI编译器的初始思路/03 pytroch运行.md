

```python
model = models.resnet50(weights=models.ResNet50_Weights.DEFAULT).cuda()
x = x.cuda()
y = model(x)
```

#### 1、调度

**而且我们之前也一直接受了代码运行在cpu上的过程，实际上这个过程也是类似的，但是这就是传统编译器的功劳了，对于AI编译器则是针对于大模型的算子来进行处理和优化等等的**



#### 2、分解

**(1) 上层算子**

PyTorch Module。

- 包括卷积、ReLU、BatchNorm 等算子
- 这是一套 **抽象接口**

**（2）后端**

**pytroch支持**

调用哪个后端的 kernel

**(3) 底层**

在 GPU 上：

- PyTorch 的卷积、矩阵乘等算子会调用 **cuDNN / cuBLAS** 等 NVIDIA 库的函数
- 这些库提供了 GPU 上高效的 kernel
- PyTorch 不需要自己写 CUDA 核心算子，只要调用库函数即可