#### Dialect 

**1.1 基本概念   **

在 MLIR 中，我们经常要定义自己的 **Dialect（方言）**，里面包含 **Types、Attributes、Operations** 等内容。

但在更高层次上，我们希望：

- 比如 **类型转换、操作优化、目标后端 Lowering** 等，可以跨 Dialect 进行处理
- **代码解耦**：使用接口而不是死绑在具体 Dialect 实现上

**1.2 interface**

我们可能在编译器优化中写了一个通用的 pass，它想要知道一个 Dialect 支持不支持某种折叠（constant folding）。这时：

- 如果每个 Dialect 都实现了一种统一的接口（比如 `ConstantFoldInterface`）
- 那优化 pass 只需要调用这个接口
- 而不用判断：这个 Op 是哪个 Dialect 的、有没有某种能力

**1.3 接口种类**

MLIR 中主要有以下几种 Interface（不只限于 Dialect Interface）：

| 附加位置     | 说明                                   |
| ------------ | -------------------------------------- |
| Operation 上 | 定义某类 Op 的行为，比如是否可以融合   |
| Dialect 上   | 定义某个方言支持的能力                 |
| Type 上      | 类型的行为，比如是否可以广播、比较大小 |
| Attribute 上 | 定义某个属性的行为                     |

**1.4 **

**（1）ableGen**

```tablegen
// MyInterfaces.td

def MyDialectInterface : DialectInterface<"MyDialectInterface"> {
  let cppNamespace = "::my_dialect";

  let methods = [
    InterfaceMethod<
      "Return true if the given type is an index type.",
      "bool", "isIndexType", (ins "Type":$type)
    >
  ];
}

```

解释：

- `DialectInterface<"名字">`
- `isIndexType(Type)`

**（2）实现**

```c++

struct MyDialectInterfaceImpl : public MyDialectInterface::Base<MyDialectInterfaceImpl> {
  MyDialectInterfaceImpl(Dialect *dialect) : Base(dialect) {}

  bool isIndexType(Type type) const override {
    return type.isInteger(32);
  }
};

```

```c++
void MyDialect::initialize() {
  addTypes<...>();
  addOperations<...>();

  addInterfaces<MyDialectInterfaceImpl>();
}

```

**（3）使用**

```c++
auto *interface = context->getRegisteredDialect<MyDialect>()
                      ->getInterface<MyDialectInterface>();
if (interface && interface->isIndexType(myType)) {
  // 这个类型是 index 类型
}

```

**1.5 mlir官方代码**

可以看看 MLIR 自带的接口定义和实现：

- `include/mlir/IR/DialectInterface.h`
- `include/mlir/Interfaces/FoldInterfaces.td`
- `lib/Dialect/LLVMIR/IR/LLVMDialect.cpp`
- `include/mlir/Interfaces/CastInterfaces.td`



#### Pass

**1、基础概念**

Pass 就是对 MLIR IR 进行“分析”或“变换”的单元。

作用包括：

- **分析**（Analysis）：统计 IR 信息，比如哪个函数用了多少参数。
- **变换**（Transformation）：把 IR 做结构替换，比如把 `mydialect.add` 换成 `arith.addi`
- **Lowering**：把高级 Dialect 降到低级 Dialect，最终可以生成 LLVM IR

**2、Pass分类**

**3、Pass的基本结构 **

**（1）最简单的Op Pass框架**

```c++
struct MySimplePass : public PassWrapper<MySimplePass, OperationPass<func::FuncOp>> {
  void runOnOperation() override {
    func::FuncOp func = getOperation();

    func.walk([](Operation *op) {
      llvm::outs() << "Found op: " << op->getName() << "\n";
    });
  }
};

```

会看到：

- `PassWrapper<YourPassClass, OperationPass<T>>` 
- `runOnOperation()` 
- `getOperation()` 

**（2）注册Pass**

```c++
std::unique_ptr<Pass> createMySimplePass() {
  return std::make_unique<MySimplePass>();
}

static PassRegistration<MySimplePass> pass("my-simple-pass", "Print all ops in a func");

```

这段代码注册了一个命令行 Pass，可以用：

```
mlir-opt test.mlir -my-simple-pass
```

