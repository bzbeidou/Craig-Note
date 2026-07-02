## TPU-MLIR补充

### 1.1 在这个项目里mean和scale是做什么的？

```py
img = img / 255.0  # 转换为 [0,1] 之间

```

那么推理时也必须用同样方式处理图像，否则预测结果就会错误。

```
img = (img - mean) * scale
```

在你这个例子中：

```
img = img * 1/255  =>  mean=0.0, scale=0.0039216
```

### 1.2 进行一次推理

```
[ONNX原始模型]
     |
     | model_transform.py
     ↓
[MLIR文件 yolov5s.mlir]
     ↓
[ONNX模型跑一次推理（用 dog.jpg）]
     ↓
[保存输出为 yolov5s_top_outputs.npz] ←── 这个是后面要对比用的

===============================

[后续量化阶段]
     |
     | model_deploy.py
     ↓
[跑量化模型推理（用 dog.jpg）]
     ↓
[把输出和 yolov5s_top_outputs.npz 对比]

```

**为什么要保存这一步的输出结果？**

**为了后续模型量化（quantization）做“精度校验”**：

后面会执行 `model_deploy.py`（量化和编译），这一步会：

1. 读取 `.mlir`（就是你刚刚转出来的模型）
2. 把它量化成 int8 精度
3. 然后用同样的 `dog.jpg` 做一次新的推理
4. 和 `.npz` 文件中保存的原始 float32 推理结果对比
5.  如果误差在合理范围，就认为“量化成功”

### 1.3 量化成int8精度是什么意思

把浮点数表示的模型（通常是 FP32）转换成使用 8 位整数（INT8）来表示和计算，以降低模型体积和加速推理。

而 **INT8（8-bit 整型）** 就是只使用 1 个字节来存储数据，取值范围是 `-128 ~ 127` 或 `0 ~ 255`，比浮点数节省很多空间。

就是要减少模型体积，提升推理速度

把模型中的参数（比如卷积核、全链接权重）从float32映射成int8

把输入输出张量也做 int8 映射（需要测试数据辅助决定范围）：

```
y = Conv2D(x, w)  →  y_int8 = Conv2D_int8(x_int8, w_int8)
```

```
ONNX → MLIR → 量化（INT8）→ bmodel（部署用）

           ↑
       model_transform.py     ← 生成 yolov5s.mlir（float）
           ↓
       model_deploy.py        ← 把 mlir 量化成 int8，并生成 yolov5s_int8.bmodel

```

### 1.4 传统编译器对比这里的Ai编译器

传统编译器（如 LLVM）是 ONNX → MLIR → LLVM IR → 机器码”，**TPU-MLIR 虽然使用了 MLIR 框架，但它不是一个通用编译器，它**不再走 LLVM IR，而是通过 **自定义 Dialect 和后端工具链，直接生成能在 TPU 芯片上运行的 bmodel（二进制模型）文件**。

 **LLVM/传统编译器流程（C/C++、Rust、Swift）**

```
源代码 → AST → LLVM IR → Machine IR → 汇编/机器码 → 芯片执行
或者说对于AI
ONNX → MLIR → LLVM IR → 汇编 → binary → 芯片（CPU/GPU）
```

**TPU-MLIR 编译流程（AI模型 → 芯片部署）**

```
ONNX / TF / Caffe
     ↓
model_transform.py
     ↓           # 转成 MLIR（包含 Shape、类型、Op）
MLIR (.mlir)
     ↓
model_deploy.py
     ↓           # 量化、调度、编码
INT8 bmodel（二进制）
     ↓
部署到 TPU 芯片
```

**没有 LLVM IR，也没有机器码**，而是生成一个 **硬件识别的“推理专用模型格式” bmodel**

