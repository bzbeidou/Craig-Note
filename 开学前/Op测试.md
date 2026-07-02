#### 测试Op的部分

**（1）经典注册上下文**

```
mlir::DialectRegistry registry;
mlir::MLIRContext context(registry);
context.getOrLoadDialect<mlir::north_star::NorthStarDialect>();
//这里就注册方言
```

**（2）构建MLIR自带的的一个帮助类`mlir::OpBuilder`**

```
1.构建Op，就是创建IR中的节点
2.提供创建位置（location）loc等信息
```

```c++
  mlir::OpBuilder builder(&context);
  auto loc = builder.getUnknownLoc();
```

在.mlir文件中每一个Op都有一个Location，表示来自哪一行。而 `builder.getUnknownLoc()` 表示创建一个“未知位置”的 `Location`。

Location 的作用：

- 错误报告的时候提示你这个 Op 是在哪创建的

例如你以后写 pass，如果某个 op 出错了，它会打印：

```c++
error: "my.op" failed at loc("test.mlir":42:13)
```

现在你用的是 `UnknownLoc`，所以调试信息是 `loc(unknown)`。



**（2）创建顶层MLIR模块 ModuleOp    本身 **

```c++
  auto module = builder.create<mlir::ModuleOp>(loc, "NorthStar");
  builder.setInsertionPointToStart(module.getBody());
```

ModuleOp 就像一个“大箱子”或“主程序函数”，你之后创建的各种操作（Op）都要放进这个盒子里。

```c++
builder.create<mlir::ModuleOp>(loc, "NorthStar");
```

参数含义解释：

- `builder.create<...>`：通过 `OpBuilder` 创建 Operation
- `<mlir::ModuleOp>`：你创建的是 MLIR 自带的标准 Op `ModuleOp`
- `loc`：这个 Op 的位置信息，通常用 `builder.getUnknownLoc()` 来生成一个测试位置
- `"NorthStar"`：这是 module 的名称，用于 debug 显示（有的版本可能不接受这个参数）

生成效果，等价于构建了如下MLIR IR

```
module {
  // 后续内容会插入到这里
}
```

**第二行：设置插入点**

```c++
builder.setInsertionPointToStart(module.getBody());
//告诉 OpBuilder：之后你创建的 Op 都插入到这个 module 的 body 开头处。
```

MLIR 的 Op 是树状结构，每个 `Operation` 可以有一个 “Block” body，而 `ModuleOp` 是有 body 的。你需要告诉 builder **往哪里插入新的 Operation**。

类比： 你建了一个文件夹（ModuleOp），接下来你要把文件（Op）加进去，就要告诉 builder：**“接下来都加到这个文件夹最上面”**。



**（3）构造两个张量类型和数据内容**

```c++
  auto f32 = mlir::Float32Type::get(&context);
  auto shape = mlir::SmallVector<int64_t>({2, 2});
  auto const_value_1 =
      mlir::SmallVector<llvm::APFloat>(4, llvm::APFloat((float)1));
  auto const_value_2 =
      mlir::SmallVector<llvm::APFloat>(4, llvm::APFloat((float)2));
  auto tensor_type_1 =
      mlir::north_star::NSTensorType::get(&context, shape, f32, 0);
  auto tensor_type_2 =
      mlir::north_star::NSTensorType::get(&context, shape, f32, 1);
```

*定义float32类型*

```c++
auto f32 = mlir::Float32Type::get(&context);
//创建一个 MLIR 中的 float32 类型
//Float32Type 是 MLIR 中内置的类型之一，代表 float
//它属于 MLIR 标准类型系统的一部分
//必须通过上下文来构造（类型信息都挂在 context 上）
```

*定义shape（形状）*

```c++
auto shape = mlir::SmallVector<int64_t>({2, 2});
//定义了一个张量的形状是 [2, 2]，即 2 行 2 列
//MLIR 使用 SmallVector 是因为它比 std::vector 更高效（常用于 MLIR 的容器）
```

*定义两个常量数据内容*

```c++
auto const_value_1 =
    mlir::SmallVector<llvm::APFloat>(4, llvm::APFloat((float)1));
auto const_value_2 =
    mlir::SmallVector<llvm::APFloat>(4, llvm::APFloat((float)2));
```

