#### 1、写kernel？

针对特定硬件（CPU、GPU、TPU、NPU 等）手工实现高效的计算内核，
 让硬件能以最高性能执行“卷积、矩阵乘、激活”等运算。

- 为什么不同的硬件需要定义自己的算子？

	因为不同硬件有不同的存储、不同的架构和计算能力（比如一次性可以计算多大的矩阵），所以需要针对不同的硬件设计不同的代码。

**（1）写卷积**

```py
y = F.conv2d(x, w, bias, stride=1, padding=1)
```

调用了一个卷积算子函数，同时pytorch也搭配cuda提供了支持，能支持跑gpu上



 **（2）底层**

NVIDIA 的 cuDNN 底层可能会写成这样（示例化简版）：

你可以看到：

- 这里是纯 C++ + CUDA 的实现；
- 手工控制内存布局、线程分配、cache；
- 一点点地优化访问方式以贴近 GPU 架构。

```cpp
__global__ void conv2d_kernel(float* input, float* weight, float* output,
                              int H, int W, int C_in, int C_out) {
    int out_y = blockIdx.y * blockDim.y + threadIdx.y;
    int out_x = blockIdx.x * blockDim.x + threadIdx.x;
    float acc = 0.0f;
    for (int c = 0; c < C_in; ++c)
        for (int ky = 0; ky < 3; ++ky)
            for (int kx = 0; kx < 3; ++kx)
                acc += input[(c * H + out_y + ky) * W + out_x + kx] * 
                       weight[((c * 3 + ky) * 3 + kx) * C_out];
    output[out_y * W + out_x] = acc;
}
```



#### 2、结

AI编译器工作就是链接顶层模型和顶层算子的关键，

**那么我们就可以针对一个上层算子将其转变为该底层kernel算子可计算的形式（包括类型、形状等）**

所以AI编译器这个转变的过程就包括了算子融合、量化映射、内存布局调整等等
