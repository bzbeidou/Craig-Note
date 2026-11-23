#### 引

我对TVM的理解是实现从高级模型（比如pytorch模型）转到针对每一个硬件指令，一次达到TVM所说的将模型应用到所有的硬件上。但是实际上并不是这样，硬件指令是肯定不能公开的，直接调用也是不可能的，那么TVM是怎么对应每个硬件实现对应的运行效果呢

**首先明确：**

- TVM最底层并不是针对每一个硬件通过指令调用，而是使用的厂商提供的一个对指令的抽象，比如Nvidia的CUDA，而不是直接用Nvidia的cuDNN算子库。
- TVM的目标主要目标并不是实现编译过程，而是找到一个针对具体硬件的效果很好地kernel，相当于一个代替人工写kernel的过程

**还是抽象，下面看具体介绍**

#### 1、TVM到底做了什么？

**假设有一个卷积层如下，在pytorch里这个仅仅是一个抽象算子**

```py
y = conv2d(x, w)
```

- **传统方式**  直接调用NVIDIA的cuDNN，cuDNN内容已经通过CUDA为不同GPU型号写好了几十种手工优化实现。
- 写了这段代码pytorch就会调用NVIDIA的cuDNN运行在GPU上

```py
torch.nn.Conv2d(...)
```



**如果说使用TVM呢？  那么TVM就不会去以来cuDNN，而是自己取寻找一种CUDA的实现形式，相当于自己写kernel**

TVM 不去依赖 cuDNN，而是：

1. 先分析算子（conv2d 的输入、输出、kernel、stride、padding 等信息）；
2. 自动生成一个计算模板（称为 **Compute DAG**）；
3. 再通过 **调度（schedule）** 去探索不同的实现方式（比如 loop 分块、向量化、线程划分、内存布局）；
4. 最终生成一段 **等价但优化后的 CUDA 或 LLVM IR 代码**。

比如它可能自动生成：

```c++
__global__ void conv2d_kernel(float* input, float* weight, float* output) {
    // 自动探索 tile 大小、共享内存、并行线程配置
}
```

然后再交给 **nvcc 编译器** 去生成最终的 GPU 二进制指令。

**这里就澄清了我们之前的一个误区，就是说TVM并不会涉及到底层的指令，而是最多到底层的上一层抽象接口，比如CUDA**



#### 2、再详解以下TVM自动生成CUDA的过程

以一个卷积层为例：

传统方式（PyTorch + cuDNN）：

```
y = torch.nn.Conv2d(x, w)
```

PyTorch 内部调用 cuDNN：

```
cudnnConvolutionForward(handle, ...);
```

 cuDNN 内部已经手工写好上百种卷积 kernel。

而 TVM 方式：

1. 导入模型：

	```
	import tvm
	from tvm import relay
	
	mod, params = relay.frontend.from_pytorch(model, ...)
	```

2. TVM 会把这个 `conv2d` 变成一个中间表示 (Relay IR → TensorIR)，比如：

	```
	for n, c, h, w in ...
	    for kh, kw in ...
	        Y[n, c, h, w] += X[n, c, h+kh, w+kw] * W[c, kh, kw]
	```

3. TVM 会自动**搜索最佳调度**：

	- tile 大小是多少？
	- block 和 thread 怎么划分？
	- 是否用 shared memory？
	- 是否向量化？

4. 生成 CUDA 源代码：

	```
	__global__ void tvm_gen_conv2d(float* input, float* weight, float* output) {
	    // TVM 根据 schedule 自动生成的 kernel
	}
	```

5. 调用 `nvcc` 编译生成 `.ptx`（GPU 汇编），再运行。

**实际上到这里TVM的功能就很清晰了，但是相比之下TPU-MLIR却显得有点落后，因为TPU-MLIR是针对于算能的芯片，却没有TVM的普适性，那么不能将TVM拓展到算能芯片吗？**

- 实际上没有这么简单，算能、寒武纪这些厂商并没有提供像CUDA这样的一个抽象层，而是提供了比原的算子库，类似于cuDNN，所以不可能通过TVM适配于算能的芯片。
- **至于TVM和TPU-MLIR之间的区别我们继续讲，区别很大，甚至大到可以说不是一个东西**
