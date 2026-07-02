
### 1、c++程序

#### （1）g++

cpu指令

**g++编译器已经全部完成**

```c++
int add(int a, int b) { return a + b; }
```

```
mov eax, edi   ; 把参数 a 放进寄存器 eax
add eax, esi   ; eax += b
ret            ; 返回
```

操作系统 + CPU



### 2、回到AI编译器

做了什么

- 读取onnx或者pytorch模型，然后做优化，然后把这些高层算子转成指令序列，如果封装好了库函数就更好了，直接可以用

硬件调用包装，开发编译器就直接用



### 3、感觉AI编译器的工作不重要了？计算的库都用c++的库给出来了，这不就是调用就行了吗？

#### （1）提供的库函数仅仅是一些积木块

但一个真实的模型（ResNet、Transformer、Diffusion...）可能有几百层、几千个算子。
**把整个模型拆解、优化，然后用库函数拼接起来**。

#### （2）算子调用库函数的方式不同，性能可能差的多

内存能不能对齐？

张量能不能拆分成更小的块并行？

调用顺序能不能避免重复搬运数据？

多个算子能不能融合



#### （3）编译器还能做更多高级优化

AI 编译器还能做很多“智能优化”，例如：

- **算子融合 (Operator Fusion)**：Conv + BN + ReLU → 一次 kernel 调用。
- **量化 (Quantization)**：把 FP32 自动转成 INT8，大幅降低计算量。
- **算子替换 (Op Rewrite)**：有些算子在 TPU 上不好跑，可以自动改写成等价的实现。
- **自动并行 (Auto Parallelism)**：在多核 TPU/GPU 上自动划分计算。



### 4、流程例子

- 识别算子（pattern）
	- 也就是从IR层（MLIR/ONNX）里检测到tpu.conv2d这些东西
- 类型/形状推导
	- 拿到NCHW，kernel KxR, S，stride, padding
- 选择计算算法
- tiling/切分（为了适配pain上内存LMEM）
	- 如果输入太大不能一次放进 LMEM，就把 N/C/H/W 切块（tile）。切块大小由 `Arch::LMEM_BYTES/LMEM_BANK_BYTES/NPU_NUM` 等计算得到。
- 数据 layout 与 pack
	- 根据硬件最优布局（比如 NCHW → NCHWc 或特殊 packed layout）
- 生成调用或指令
