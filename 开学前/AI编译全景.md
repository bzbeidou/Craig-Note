#### 1、AI编译器开发   ----把大模型（llama2等）变成能高效运行在芯片（GPU/FPGA/ASIC）上的形式

**输入：**

​	模型+芯片硬件

**输出：**

  针对芯片优化的模型程序（可执行代码）

**设计两个部分（AI编译器+Runtime）可以拆分成5个阶段：**

*前四个部分属于AI编译器内容，最后运行是属于Runtime设计阶段*

**（1）前端（Front-End）就是规范统一化模型 ** 

  输入模型格式多样（PyTorch、TensorFlow、ONNX）

  这一阶段目的就是统一成一种内部表示（IR， Intermediate）

**我现在用的ONNX就是一个中间格式，其中MatMul、Reshape、Add等操作都被表示为ONNX图中的节点**

**（2）中端（Middle-End）  图优化与变换**

比如图算子融合（如 Conv+BN+ReLU → 一个块）

**（3）后端（Back-End）  把优化后的算子转换成指令** 

把算子转换成指令

**（4）代码生成**

**（5）运行**



#### 2、部分术语什么是 Relay、TensorIR、MLIR、TVM、XLA？

说人话：TVM是编译器、XLA是编译器、MLIR是工具，Relay、TensorIR是TVM的两个阶段

  **(1)TVM**    Tensor Virtual Machine

TVM就是AI编译器，但是他也提供一些设计编译器的方法

TVM支持导入pt、tf、onnx等模型，最终生成各种平台代码（比如CUDA、LLVM等）

Relay和TensorIR是TVM直接作为编译器使用时的两个阶段

Relay就像是TVM中的onnx，就是把输入模型转换成统一格式，但比onnx格式更复杂

TensorIR作用暂时理解为“转换成指令”

**(2)MLIR**  Multi-Level Intermediate Representation多层中间表示

  如其名，用于构建多层IR（从模型层到硬件层）

**(3)XLA**  

也是AI编译器，但是只能用于解析Tensorflow模型



#### 
