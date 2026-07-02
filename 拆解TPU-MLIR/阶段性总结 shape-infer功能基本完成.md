### 1、注册Pass的实际部分

也就是说注册Pass这个工作究竟是怎么做的，包括能够识别`--shape-infer`这种命令等等

**实际上在自动生成的部分：**

```c++
inline void registerMyTestPass() {
  ::mlir::registerPass([]() -> std::unique_ptr<::mlir::Pass> {
    return createMyTestPass();
  });
}
```

### 2、注册Dialect的部分

这部分需要创建一个`DialectRegistry`对象

```c++
registry.insert<mlir::func::FuncDialect,mlir::scf::SCFDialect, mlir::arith::ArithDialect, top::TopDialect>();
```

### 3、Dialect部分 涉及Op的注册

`initialize()`

**`addOperations`**

```c++
void TopDialect::initialize() {
  llvm::errs() << "Registering TopDialect ops...\n";
  addOperations<
#define GET_OP_LIST
#include "tpu_mlir/Dialect/Top/IR/TopOps.cpp.inc"
  >();
}
```

