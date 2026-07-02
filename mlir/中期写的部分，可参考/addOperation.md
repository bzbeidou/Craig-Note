### 做什么

告诉当前的Dialect中有这个方法

**结合上tablegen的具体内容如下**

- 在.td文件里定义了一个Op,比如AddOp
- 前Dialect的initialize()方法里 `addOperation<AddOp>()`

**（1）dialect,这是tablegen**

```
// MyDialect.h
#include "mlir/IR/Dialect.h"

namespace mydialect {
class MyDialect : public mlir::Dialect {
public:
  explicit MyDialect(mlir::MLIRContext *ctx)
      : Dialect(getDialectNamespace(), ctx, mlir::TypeID::get<MyDialect>()) {
    initialize();
  }

  static llvm::StringRef getDialectNamespace() { return "mydialect"; }

  void initialize();
};
} // namespace mydialect

```

**（2）算子**

```
// MyOps.td
def MyDialect : Dialect {
  let name = "mydialect";
  let cppNamespace = "::mydialect";
}

def My_AddOp : Op<MyDialect, "add", [NoSideEffect]> {
  let summary = "Simple add operation";
  let arguments = (ins F32:$lhs, F32:$rhs);
  let results = (outs F32:$output);
}

```

**（3）注册**

```
// MyDialect.cpp
#include "MyDialect.h"
#include "MyOps.h" // TableGen 生成的头文件

using namespace mydialect;

void MyDialect::initialize() {
  addOperations<
    My_AddOp // 注册我们定义的加法算子
  >();
}
```



**（4）测试**

```
module {
  %0 = mydialect.add %lhs, %rhs : f32
}

```

