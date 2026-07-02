#### 1 Context

```mlir
func @add_func(%arg0: f32, %arg1: f32) -> f32 {
  %0 = arith.addf %arg0, %arg1 : f32
  return %0 : f32
}
```

具体代码如下：

```c++
#include "mlir/IR/MLIRContext.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinOps.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/IR/Verifier.h"
#include "mlir/IR/AsmState.h"
#include <iostream>

using namespace mlir;

int main() {
  MLIRContext context;
  context.getOrLoadDialect<func::FuncDialect>();
  context.getOrLoadDialect<arith::ArithDialect>();

  OpBuilder builder(&context);
  auto loc = builder.getUnknownLoc();
  auto module = ModuleOp::create(loc);

  auto f32Type = builder.getF32Type();
  auto funcType = builder.getFunctionType({f32Type, f32Type}, {f32Type});

  auto funcOp = builder.create<func::FuncOp>(loc, "add_func", funcType);

  module.push_back(funcOp);

  Block *entryBlock = funcOp.addEntryBlock();  
  builder.setInsertionPointToStart(entryBlock); 
  auto arg0 = entryBlock->getArgument(0);
  auto arg1 = entryBlock->getArgument(1);
  auto addOp = builder.create<arith::AddFOp>(loc, arg0, arg1);
  builder.create<func::ReturnOp>(loc, addOp.getResult());     

  if (failed(verify(module))) {
    std::cerr << "Module verification failed!" << std::endl;
    return 1;
  }

  module.print(llvm::outs());
  std::cout << std::endl;

  return 0;
}

```



#### 2 使用

**（1）读**

`MLIRContext &context;`这个创建Context实例的后面实现的。而且读取的过程中会看MLIRContext里是不是定义了当前读取到的Dialect，如果发现有注册.

**链接.mlir文件和IR之间的桥梁**

**（2）修改**

-  **OpBuilder** 
- **OpBuilder 的构造** 

```c++
OpBuilder builder(&context);
builder.create<tpu::Conv2DOp>(...);
```

#### 3、内部

定义清楚，成员变量用来存储，我们要像使用Dialect或者Type，

**全局只有一个实例**

**查找Dialect这种元信息比较统一了**

```c++
class MLIRContext {
public:
  MLIRContext();
  ~MLIRContext();

  // 一些公有接口
  Dialect *getOrLoadDialect(StringRef name);
  template <typename T> T *getOrLoadDialect();

  DiagnosticEngine &getDiagEngine();

private:

  llvm::StringMap<std::unique_ptr<Dialect>> loadedDialects;

  StorageUniquer typeUniquer;

  StorageUniquer attrUniquer;

  llvm::StringMap<OperationName> registeredOperationNames;

  DiagnosticEngine diagEngine;

  bool threadingEnabled;
};

```

