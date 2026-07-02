# 一、8.2残留问题解决

## 1、pattern的种类



## 2、 ModuleOp这些的存在形式，也就是parseSourceFile这个的作用，之前一直存在这个问题

`parseSourceFile` 是 MLIR 自带的函数，用来**读取 `.mlir` 文件的源码文本，并将其解析成 IR**

**作用：把.mlir文本变成真正的IR结构(ModuleOp等)**

**例，MLIR文件代码如下**

```
module {
  func.func @add(%a: i32, %b: i32) -> i32 {
    %0 = arith.addi %a, %b : i32
    return %0 : i32
  }
}
```

```c++
mlir::MLIRContext context;
context.loadDialect<mlir::func::FuncDialect, mlir::arith::ArithDialect>();

// 打开文件
auto file = mlir::openInputFile("example.mlir");
if (!file) { /* 错误处理 */ }

// 解析成 ModuleOp（一个 IR 模块）
mlir::OwningOpRef<mlir::ModuleOp> module = mlir::parseSourceFile<mlir::ModuleOp>(
    file->getBuffer(), file->getBufferIdentifier(), &context);
```

它内部做了这些事：

1. **词法分析（Lexing）**：把文本拆成 Token
2. **语法分析（Parsing）**：识别 `module`、`func.func` 等结构
3. **根据 Dialect 注册信息**：
	- 找到 `func.func` 对应的 `FuncOp`
	- 找到 `arith.addi` 对应的 `AddIOp`
4. **调用每个 Op 的 `parse()` **
	- 每个 `Op` 都有一个 `::parse()` 静态方法
	- MLIR 调用这个方法解析参数、返回值、类型等
5. **用 `Operation::create(...)` 分配内存,这部分先不用看**

## 3、什么是MLIR的IR结构  就是有各个Op对象组成的一个整体嵌套结构

- 加入有以下文本

```
module {
  func.func @add(%a: i32, %b: i32) -> i32 {
    %0 = arith.addi %a, %b : i32
    return %0 : i32
  }
}

```

- 那么这段IR的结构可以如下画出来

```
ModuleOp
└── Region
    └── Block
        └── FuncOp @add
            └── Region
                └── Block
                    ├── %0 = arith.addi ...
                    └── return ...

```

每一个 **方块**，比如 `FuncOp`、`arith.addi`、`return`，**都是一个 Operation 对象的实例**，也就是 IR 的组成部分。

```c++
class Operation {
  SmallVector<Region, 1> regions;
  Block *block;                 // 所在 block
  Operation *next;             // 同一个 block 中下一个 op
  Operation *prev;             // 上一个 op
}
```

# 二、MLIR内容复习

## 1、MLIRContext与DialectRegister

**是两个核心类**

```c++
mlir::DialectRegistry registry;
tpu_mlir::registerAllDialects(registry);

// 构建 MLIR 上下文时，把 registry 传进去
mlir::MLIRContext context(registry);

```

`DialectRegistry` 就是用来 **收集和注册这些方言的地方**,就是一个词典，创建这个的实例对象，然后第二行的内部实际上是用了它的insert方法，只不过进行了一个封装

```c++
registry.insert<tpu::TPUDialect>();
registry.insert<top::TopDialect>();
// 还有其他可能的 Dialect
```

这样就把已经定义好的Dialect收集好了，就是传入的泛型是Dialect的类定义而已，然后都存在register实例对象里

MLIRContext也比较类似，就是MLIR的上下文，**Operation、Type、Attribute、Dialect 等等**都是在这个上下文背景之下

需要把 DialectRegistry 的内容 **传入 MLIRContext**，它才会把这些 Dialect 真正地“加载进来”，也就是`mlir::MLIRContext context(registry); `的作用，把register的内容加载进来了。

`DialectRegistry`：只是一个“待注册列表”

`MLIRContext`：才是实际注册、管理、使用 Dialect 的地方，

**区分一下Pass的注册问题，Pass相当于不是在MLIRContext里的，而是由PassManager来管理的，这两个可以是一个同级的概念**

但是在使用**PassManager时需要把context传进去，但是只是用来用其中的一些工具**

```c++
mlir::MLIRContext context;
mlir::PassManager pm(&context);  // 把 context 传进去，但不是由 context 管理的

pm.addPass(mlir::createCanonicalizerPass());
pm.addPass(mlir::createCSEPass());

// 解析 IR，运行 Pass
mlir::ModuleOp module = ...; // 从文件中 parse
if (failed(pm.run(module))) {
  llvm::errs() << "Failed to run passes\n";
}

```

# 三、c++

## 1、创建对象的问题

java中必须用new在堆上创建对象，但是c++可以不用new 在栈上创建对象或者用new在堆上创建对象

```c++
MyClass a;               // 栈上创建，自动析构
MyClass* p = new MyClass();  // 堆上创建，需要手动 delete（或智能指针,一般使用智能指针）
```

这里的`MyClass a`就会自动调用MyClass的构造函数然后在栈上创建，如果有参数可以MyClass a()这样来创建

残留问题

Operation类的源码这部分的源码需要再看一下