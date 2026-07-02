## 1、Top和Tpu之下的文件内容对比

### 1.1 interface

| 位置               | 用途                                                         | 粒度             | 举例                                   |
| ------------------ | ------------------------------------------------------------ | ---------------- | -------------------------------------- |
| **Top/Interfaces** | 定义在 **高层 IR**（top dialect）上的接口，更多是通用的，不依赖具体芯片 | 比较抽象         | Op 的 shape 推导接口、广播规则接口等   |
| **Tpu/Interfaces** | 针对 **TPU Dialect** 的接口实现，并且和具体芯片绑定          | 更具体、接近底层 | 不同芯片的指令生成、参数准备、执行配置 |

在Tpu的Interfaces之下支队具体芯片划分了不同的实现内容

**BM1684** → 算能早期 TPU 芯片架构

**BM1684X** → 升级版架构（指令集、硬件特性有变化）

**CV18xx** → CVITEK类芯片架构

**Common** → 这些平台的公共实现（复用逻辑）

**内容**

对于Tpu来说会实现几个关键接口

例如：

- **`inference()` / `codegen()`** → 推理计算 & 生成硬件执行代码
- **`dyn_codegen_global()` / `dyn_codegen_local()`** → 针对动态 shape 的代码生成
- **`isSupport()`** → 判断该硬件是否支持此 Op 的某种模式

因此会看到

```
BM1684/Add.cpp   → AddOp 在 BM1684 芯片上的生成逻辑
BM1684/Arg.cpp   → ArgOp 在 BM1684 芯片上的生成逻辑
BM1684X/Add.cpp  → AddOp 在 BM1684X 芯片上的生成逻辑
...
Common/Add.cpp   → AddOp 各芯片通用的基础逻辑
```

### 1.2 canonicalize

存放各种 Op 的**模式优化（Pattern Rewrite）规则**，通常和硬件无关，是 MLIR 的 IR 级别优化。

### 1.3 调用流程

- **top dialect** (高层 IR)

- **lower 到 tpu dialect**

- **调用 Tpu/Interfaces 里对应芯片的接口**，生成硬件执行代码（如 bmodel）

- **部署到芯片运行**



## 2、TPU

### 2.1 Add.cpp

#### 2.1.1 LogicalResult tpu::AddOp::init

**功能**：初始化这个 Op 运行所需的公共资源

**可能做的事**：

- 读取 Op 的属性（scale、offset、是否带 ReLU 等）
- 分配中间 buffer（如果推理/训练都用到）
- 准备量化参数

**作用阶段**：编译器在 codegen 前或运行准备阶段调用



`AddOp::getInputs()`返回的就是一个 **`operand_range` 对象**，这里保存了AddOp的所有输入operands



#### 3.1.2 void tpu::AddOp::deinit

**功能**：释放 `init()` 分配的资源

**可能做的事**：

- 释放 buffer
- 清空中间状态

**作用阶段**：编译或运行结束后调用



**computer_broadcast_shape**

在lib/Interfaces/ShapeInterface.cpp之下，就是在做一个多个输入的输出形状广播推导

逻辑如下

它就是在做一个 **多输入张量的广播 shape 推导**，逻辑是：

1. 先拿第一个输入的 shape 作为初始输出 shape
2. 再和后续的每个输入 shape 按广播规则对齐（从最后一维往前比）
3. 得到最终广播后的输出 shape

- `llvm::SmallVector<int64_t>`类似 `std::vector<int64_t>`，只是优化了内存分配

- `op->getNumResults()`MLIR 的 `Operation` 有多少个输出（这里要求只能有一个输出）。
- **`op->getOperand(i)`** 取第 `i` 个输入（operand）。
- **`module::getShape(...)`** 从某个 `mlir::Value`（张量）中拿它的 shape，返回 `SmallVector<int64_t>`。
- **`llvm::reverse`**
	 反向迭代，比如 shape `{4, 1, 3}` 反过来是 `{3, 1, 4}`。
- **`llvm::zip_longest(a, b)`**
	 像 Python `itertools.zip_longest` 一样，把两个序列按位置配对，不够长的用 `None` 填。

#### 3.1.3 LogicalResult tpu::AddOp::inference

`inference()` 是 **CPU 端 Add/减法运算的执行逻辑**，它会：

1. 先根据广播规则计算输出 shape，并更新到 MLIR 模型中。
2. 为两个输入数据构造 `Binary` 算子对象（CPU 推理类）。
3. 根据系数 (`getCoeff()`) 决定执行 **加法** 还是 **减法**。
4. 支持可选的 **ReLU 激活**。
5. 将 `Binary` 对象存入 `InferenceParameter.handle`，供后续使用或释放。





#### 3.1.4 LogicalResult tpu::AddOp::BackwardH  属于算子切分的接口

**（1）首先介绍一下这里的核心任务**

一个大的 tensor（比如一张 224x224 的图片）不能一次性全部放进芯片的片上内存（SRAM），所以必须 **切成小块（tile）** 来计算。这就是算子切分，切分的是数据，而且一般是从输出tensor开始切分的，但是算子需要知道：**当输出的某一部分被切分出来计算时，输入 tensor 对应的哪一部分需要拿出来？**

**这就是这个接口的作用，不是实现算子切分的，而是找到被切分的输出tensor部分对应的输入部分**

于是就需要这两个接口：

- `BackwardH`：当我们决定在 **高度 H** 上切出某一块输出时，如何反推需要的输入高度范围？
- `BackwardW`：当我们决定在 **宽度 W** 上切出某一块输出时，如何反推需要的输入宽度范围？



**因此：这个接口核心任务就是给定输出切片的起点和长度，计算对应的输入切片的起点和长度。 **



**这里又分两个情况：**

**简单情况（逐元素算子 Add）**：

- 输出和输入是一一对应的；
- 所以输入切片和输出切片完全一样。

**复杂情况（带 stride 的算子）**：

- 如果有 stride（比如卷积的步幅、early stride 优化），那输入区域就比输出区域“更大”。
- 这时就要用 `BackwardH/BackwardW` 把输入切片范围放大。



**举个简单案例：假设有一个Add算子：**

```
输入A: [1, 3, 8, 8]   // NCHW 格式，H=8, W=8
输入B: [1, 3, 8, 8]
输出C: [1, 3, 8, 8]
```

现在需要再H维做切分，芯片一次只能算 4 行

所以输出被分成两块：

- 第一块：`out_idx = 0, out_slice = 4`
- 第二块：`out_idx = 4, out_slice = 4`

调用 `BackwardH`：

- 默认情况下（逐元素操作，没有 stride）：
	- 输入切片 = 输出切片
	- 第一块输入：`in_idx = 0, in_slice = 4`
	- 第二块输入：`in_idx = 4, in_slice = 4`

这就告诉编译器，如果要算输出的前 4 行，那只需要取输入的前 4 行；如果我要算输出的后 4 行，就取输入的后 4 行。



**如果有 early stride 的情况**

比如 `early_stride_h = 2`（意思是输入每两行对应输出一行）。

- 输出切片：`out_idx = 0, out_slice = 4`
- 输入切片就要放大：
	- `in_idx = 0 * 2 = 0`
	- `in_slice = 4 * 2 = 8`

也就是说，输出的 4 行，其实需要从输入的 8 行里计算出来。

#### 3.1.5 LogicalResult tpu::AddOp::BackwardW



#### 3.1.6 LogicalResult tpu::AddOp::LocalGenSupport

**功能**：判断这个 Op 是否支持 **局部生成**（Local Generation）

**背景**：

- TPU 编译时会把大 tensor 按 tile 切片到局部 SRAM
- 有些算子支持局部处理，有些必须全局处理

**可能返回**：

- `success()` → 支持本地分块计算
- `failure()` → 必须全局计算

#### 3.1.7 void tpu::AddOp::DumpQuantAgnosticAttrs

`DumpQuantAgnosticAttrs()` **不参与计算**，只做 **属性打印/调试**。

**核心目的**：打印 **AddOp 的属性信息**，便于调试或日志分析，而且这里还把量化的属性单独拿出来了，也就是单独打印

**逻辑解析**

1. **遍历所有属性**：
	- `getOperation()->getAttrs()` 得到一个字典，键是属性名，值是属性内容。
	- 对每个属性：
		- 如果是量化相关的就跳过；
		- 否则打印 `name=value`。
2. **处理 rshifts / multipliers**：
	- 这两个属性是量化参数：
		- `rshifts` 是右移量（量化缩放的一部分）；
		- `multipliers` 是乘数（量化缩放的一部分）。
	- 如果这些数组全是 “默认值”（0 或 1），就忽略；
	- 否则打印数组长度，表示非平凡量化存在。

假设一个 AddOp 在 .mlir 文件里：

```mlir
%3 = tpu.add %0, %1 {
  do_relu = true,
  do_early_stride = false,
  rshifts = [0, 0, 0],
  multipliers = [1, 1, 1]
} : tensor<1x3x8x8xf32>
```

调用 `DumpQuantAgnosticAttrs(os)` 后的输出可能是：

```
do_relu=true; do_early_stride=false;
```

解释：

- `do_relu` 和 `do_early_stride` 是量化无关属性 → 打印出来
- `rshifts` 全是 0 → 忽略
- `multipliers` 全是 1 → 忽略

如果 `rshifts = [0,1,0]`，输出就会多一条

```
do_relu=true; do_early_stride=false; rshifts_len=3;
```



#### 3.1.8`void tpu::AddOp::assign_sec_info`
**核心目的：**

实际部署时，一个大的 tensor 要被分片到 **芯片本地 SRAM** 中才能计算。
 但是 Add 有两个输入，如果两个输入在切分时对不上，就会计算错误。**所以这里要做对齐切片和记录切片信息两个工作内容**

**对齐切片**：保证输入0、输入1、输出在 NCDHW 各个维度上的切分是一致的；

**记录切片信息**：把最终的切分参数写到 `sec_info`，后面 LocalGen 调度时会用到。



#### 3.1.9 `void tpu::AddOp::assign_fw_param`
- **功能**：为 AddOp 分配 **firmware 参数**
- **区别于 sec_info**：
	- sec_info 是“在哪执行 + 数据分区”
	- fw_param 是“执行时需要的参数结构体”，比如：
		- 数据指针
		- 常量参数
		- 激活/量化开关
- **可能会填充到硬件固件用的 struct 中**

#### 3.1.10 `ArrayAttr tpu::AddOp::getIndexingMaps`
- **功能**：返回 MLIR `linalg` Dialect 里常用的 **索引映射数组**
- **作用**：
	- 告诉编译器如何将循环迭代变量映射到 tensor 索引
	- 用于 affine/linalg 优化、pattern 匹配
- **对 Add 来说**：
	- 输入和输出 shape 通常一致
	- 索引映射也比较直接（一对一映射）

#### 3.1.11 `bool tpu::AddOp::support_multi_core`
- **功能**：判断 AddOp 是否支持多核并行
- **可能条件**：
	- 是否可以按某个维度切片
	- 数据依赖是否允许多核同时运行
	- 硬件资源是否足够
- **返回值**：
	- `true` → 支持多核（更高并行度）
	- `false` → 必须单核（可能有依赖或局限）
