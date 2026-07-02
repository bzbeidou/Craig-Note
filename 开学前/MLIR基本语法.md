#### MLIR学习

**文件基本结构**

MLIR 文件通常包含三部分：

- **Module**：是顶级的容器，用来组织所有的函数、全局变量等。
- **Func**：函数定义，表示一个独立的计算单元。
- **Operations**：每个函数中执行的操作，表示计算任务。

**基本的MLIR语法**

#### (1) **Module（模块）**

模块是 MLIR 文件的顶层元素，用于封装所有的函数、变量和其它资源。

```
module {
  // 这里是模块中的内容
}

```

#### (2) **Func（函数）**

函数定义用 `func` 关键字来表示。函数可以接收参数，返回值以及内部实现的操作。

```
func @function_name(%arg1: tensor<1xf32>, %arg2: tensor<1xf32>) -> tensor<1xf32> {
  // 函数体
}
```

`@function_name` 是函数的名字。

`%arg1` 和 `%arg2` 是输入参数，每个参数后面跟着类型（如 `tensor<1xf32>`）。

`-> tensor<1xf32>` 表示函数的返回类型。

#### (3) **Operation（操作）**

操作表示具体的计算任务。每个操作通常有一个操作名（例如 `Add`、`Mul`）、操作数（输入）和结果（输出）。

```
%result = add %arg1, %arg2 : tensor<1xf32>
```

`%result` 是该操作的结果（一个张量）。

`add` 是操作类型（加法）。

`%arg1` 和 `%arg2` 是输入参数。

`tensor<1xf32>` 是参数和结果的类型，表示 1 维浮点数张量。

#### (4) **类型（Types）**

在 MLIR 中，类型是非常重要的，它们指定了每个操作的输入和输出的数据结构。常见类型包括：

- `tensor<NxMxDxf32>`：表示一个具有 `N` 行、`M` 列、`D` 深度的浮点数张量。
- `i32`：表示 32 位整数。
- `f32`：表示 32 位浮点数。

```
%tensor_result = "onnx.add"(%tensor1, %tensor2) : (tensor<4x4xf32>, tensor<4x4xf32>) -> tensor<4x4xf32>
```

这里 `"onnx.add"` 是一个自定义的操作，它接受两个形状为 `(4x4)` 的浮点数张量，并返回一个相同形状的张量。

#### (5) **定位信息（Loc）**

MLIR 允许为每个操作添加源代码位置标记，以便于调试。位置标记通常包含文件名、行号和列号，但在实际 MLIR 文件中，您可能会看到类似以下的标记：

```
loc("file_name", 10, 20)

```

这表示操作来自于文件 `file_name`，位置位于第 10 行第 20 列。`loc` 用于帮助追踪错误和调试。

#### (6) **常见操作示例**

**Add**：加法操作

```
%result = add %a, %b : tensor<1xf32>
```

**MatMul**：矩阵乘法

```
%result = matmul %matrixA, %matrixB : tensor<2x3xf32>, tensor<3x2xf32> -> tensor<2x2xf32>
```

**Reshape**：重塑张量形状

```
%reshaped = reshape %input_tensor into tensor<4x4xf32> : tensor<16xf32>
```

**Transpose**：转置操作

```
%transposed = transpose %input_tensor : tensor<4x3xf32> -> tensor<3x4xf32>
```

**Slice**：切片操作

```
%slice = slice %input_tensor[0:3, 0:3] : tensor<4x4xf32> -> tensor<3x3xf32>
```





**文件实例：**

```
module {
  func @matmul(%A: tensor<2x3xf32>, %B: tensor<3x2xf32>) -> tensor<2x2xf32> {
    %0 = matmul %A, %B : tensor<2x3xf32>, tensor<3x2xf32> -> tensor<2x2xf32>
    return %0 : tensor<2x2xf32>
  }
}

```



**%是什么？**

**`%` 表示变量**：在 MLIR 中，任何以 `%` 开头的字符串都是一个变量（可以是张量、标量或中间计算的结果）。例如，`%result`、`%a`、`%tensor1` 等都代表变量。

**SSA 形式**：`%` 表示的变量遵循**静态单赋值**（SSA）（Static Single Assignment）形式，也就是说，每个变量只能在它的定义位置被赋值一次。之后任何对这个变量的引用都只能使用其计算出的结果，而不能被修改。这使得程序分析和优化变得更加简单。

**变量与操作的关系**：这些变量通常是在操作中生成的。例如，在矩阵乘法中，`%result` 变量可能表示矩阵乘法操作的输出，`%A` 和 `%B` 则是输入的矩阵。

**%后面变量名是可以任意取的，但是最好有一定含义，一般通过_连接，而不是大小写**



**mlir中的数据类型**

**mlir中的全部数据都是tensor类型，区别在于 tensor 的形状和元素类型**

**元素类型（Data Type）**：这个部分决定了张量元素的类型，比如：

- `f32`：32 位浮点数
- `i32`：32 位整数
- `bool`：布尔类型
- `index`：索引类型（通常用于数组的大小、偏移等）

这些元素类型是对张量中每个单元的数据类型的描述。

**形状（Shape）**：它描述了张量的维度（例如，几维、每一维的大小）。例如：

- `tensor<f32>`：表示一个零维的浮点数标量。
- `tensor<4xf32>`：表示一个包含 4 个元素的 1 维张量（向量）。
- `tensor<2x3xf32>`：表示一个 2 行 3 列的 2 维矩阵。



**@符号是什么？   用来定义函数或者调用外部函数    可以把他看作是函数名前的一个固定符号**

```mlir
func @my_function(%arg1: i32, %arg2: f32) -> i32 {
  // function body
}

```

在这里，`@my_function` 是函数 `my_function` 的符号，表示它是一个全局定义的函数, 。

```
%result = call @external_func(%input) : (i32) -> f32
```

`@external_func` 代表一个外部定义的函数, 用@来调用了这个名为external_func的非本模块内定义的外部函数。