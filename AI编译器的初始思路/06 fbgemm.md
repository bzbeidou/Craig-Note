#### 引

在获取resnet50的量化模型的时候，看到了这种量化方式，fbgemm

```c++
model_fp32.qconfig = torch.ao.quantization.get_default_qconfig('fbgemm')
```



#### 1、fbgemm是什么

facebook针对x86架构开源的量化内核库（quantized kernel library）

那么pytorch的量化推理在cpu上的底层计算，就是通过fbgemm来完成的。

#### 2、底层

FaceBook就针对这种公开的指令集写出最优的指令调度方案，就形成了kernel库fbgemm



#### 3、特定芯片的优化



| 厂商                         | 库名                 | 平台       | 说明                          |
| ---------------------------- | -------------------- | ---------- | ----------------------------- |
| **Facebook**                 | **fbgemm**           | x86 CPU    | PyTorch CPU 量化后端          |
| **ARM**                      | **QNNPACK**          | ARM CPU    | 移动端优化库                  |
| **Google**                   | **XNNPACK**          | 通用 CPU   | Chrome / TensorFlow Lite 使用 |
| **Intel**                    | **oneDNN (MKL-DNN)** | Intel CPU  | 自家优化库                    |
| **NVIDIA**                   | **cuDNN**            | GPU        | CUDA 优化卷积库               |
| **Apple**                    | **BNNS / MPS**       | M 系列芯片 | Metal 底层加速                |
| **华为**                     | **ACL / AscendCL**   | 昇腾 NPU   | 自家底层算子库                |
| **算能 / 寒武纪 / 比特大陆** | **TPU 库**           | 自研芯片   | 只提供 C++ 接口，底层算子封闭 |