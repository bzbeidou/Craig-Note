

## 1. MLIR优化框架

### 1.1 优化

- 常量折叠（`x + 0 -> x`）
- 死代码消除（不再被使用的变量删掉）
- 循环展开、内联函数、表达式简化……

**Pass：**在 MLIR 中， **Pass 就是对 IR 执行的某个优化逻辑**，也就是优化是目标，Pass是我们实现优化的过程。

**优化举例：**

优化前的 IR：

```mlir
%0 = arith.constant 0 : i32
%1 = arith.addi %x, %0 : i32

```

CSE/ConstantFold Pass 优化后：

```
%1 = %x  // 折叠掉了“加0”这个无意义的操作
```

### 1.2 MLIR常用优化Pass

| Pass 名称      | 功能说明                                                   |
| -------------- | ---------------------------------------------------------- |
| `canonicalize` | Canonicalization：进行标准化模式匹配（Pattern-based 重写） |
| `cse`          | Common Subexpression Elimination，合并重复表达式           |
| `inline`       | 内联函数体（`func.call` -> 函数体替换）                    |
| `symbol-dce`   | 删除未使用的符号（函数、全局变量等）                       |
| `loop-unroll`  | 展开循环体，加快执行（主要是 `scf.for`）                   |

**1.2.1 Canonicalize**

**作用**：对 IR 进行模式匹配式的重写，把表达式转成“标准形式”，简化逻辑。

**举例：**

```
%0 = arith.constant 0 : i32
%1 = arith.addi %x, %0 : i32
```

**优化：**

```
%1 = %x
```

**1.2.2 CSE**

**作用**：找到并合并重复的计算。

**举例：**

```
%1 = arith.addi %a, %b : i32
%2 = arith.addi %a, %b : i32
```

**优化：**

```
%1 = arith.addi %a, %b : i32
%2 = %1
```

**1.2.3 Inlining**

**作用**：将函数调用点替换为函数体本身，减少函数调用开销。

**举例：**

```
func.func @add(%a: i32, %b: i32) -> i32 {
  %0 = arith.addi %a, %b : i32
  return %0 : i32
}

%1 = func.call @add(%x, %y) : (i32, i32) -> i32

```

**优化：**

```
%0 = arith.addi %x, %y : i32
%1 = %0
```





## 2. Pass编写基础

### 2.1 

MLIR 提供的  `canonicalize`, `cse`, `inline` 等Pass处理的是对应的dialect，无法识别自定义dialect的内容。

定义了一个自己的 dialect `toy.add`，表示张量加法，那么：

```
%1 = toy.add %a, %b
%2 = toy.add %a, %b
```

自带的 `cse` 不认识 `toy.add`，不会处理它, 需要自己写 Pass，去识别、重写这些逻辑。

### 2.2 构建方式

MLIR 中的 Pass 是一个 **C++ 类**，继承了 `mlir::PassWrapper`并重写其 `runOnOperation()` 方法：

```c++
struct MyPass : public mlir::PassWrapper<MyPass, mlir::FunctionPass> {
  void runOnFunction() override {
    // 这里写具体的优化逻辑
  }
};
```

Pass 可以有几种作用域：`OperationPass<FuncOp>`：操作函数体,`ModulePass`：操作整个模块等等



### 2.3 编写Pass

写一个自定义 Dialect `mydialect`，定义一个 Op：`mydialect.addi`，它模拟 `x + y` 的整数加法。然后编写一个 Pass，优化掉其中 `x + 0` 或 `0 + x` 的加法。

#### 2.3.1 定义Dialect和Op（前面部分教程）

#### 2.3.2 准备测试用的 IR 文件

```
module {
  func.func @test(%arg0: i32) -> i32 {
    %c0 = arith.constant 0 : i32
    %sum = mydialect.addi %arg0, %c0 : i32
    return %sum : i32
  }
}
```

#### 2.3.3 编写自定义Pass 

```c++
#ifndef MY_PASS_H
#define MY_PASS_H

#include "mlir/Pass/Pass.h"

namespace mlir {
std::unique_ptr<OperationPass<func::FuncOp>> createRemoveMyAddZeroPass();
}

#endif
```

 `lib/MyPass.cpp`

```c++
#include "MyPass.h"
#include "mydialect/MyOps.h.inc"

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/IR/Builders.h"
#include "mlir/Pass/Pass.h"

using namespace mlir;

namespace {

struct RemoveMyAddZeroPass
    : public PassWrapper<RemoveMyAddZeroPass, OperationPass<func::FuncOp>> {
  void runOnOperation() override {
    func::FuncOp func = getOperation();
    OpBuilder builder(func.getContext());

    func.walk([&](mydialect::AddIOp op) {
      Value lhs = op.getLhs();
      Value rhs = op.getRhs();

      auto isZeroConst = [](Value val) -> bool {
        if (auto cst = val.getDefiningOp<arith::ConstantOp>()) {
          if (auto attr = cst.getValue().dyn_cast<IntegerAttr>())
            return attr.getValue().isZero();
        }
        return false;
      };

      if (isZeroConst(lhs)) {
        op.replaceAllUsesWith(rhs);
        op.erase();
      } else if (isZeroConst(rhs)) {
        op.replaceAllUsesWith(lhs);
        op.erase();
      }
    });
  }
};

} // namespace

std::unique_ptr<OperationPass<func::FuncOp>> mlir::createRemoveMyAddZeroPass() {
  return std::make_unique<RemoveMyAddZeroPass>();
}

```

#### 2.3.4 注册并运行 Pass

```c++
#include "mlir/Support/MlirOptMain.h"
#include "mlir/InitAllDialects.h"
#include "mlir/InitAllPasses.h"

#include "mydialect/MyDialect.h"
#include "MyPass.h"

int main(int argc, char **argv) {
  mlir::DialectRegistry registry;
  registry.insert<mlir::func::FuncDialect, mlir::arith::ArithDialect, mlir::mydialect::MyDialect>();

  mlir::registerAllPasses();
  mlir::PassRegistration<mlir::RemoveMyAddZeroPass>();

  return mlir::MlirOptMain(argc, argv, "My Pass Tool\n", registry);
}

```

### 2.3.5 优化

```
./bin/my-opt test/test.mlir -remove-my-add-zero
```

输出 IR 会把 `mydialect.addi %x, %c0` 删除



## 3 Rewrite Pattern

### 3.1 基本定义

 `RewritePattern` 系统，这是MLIR 编写 Pass 最推荐的优化方式，是对 IR 中某种“模式”的识别和替换的描述。由 MLIR 提供的机制自动完成匹配、替换、遍历等工作。

### 3.2 具体步骤

#### 3.2.1 定义Pattern

`OpRewritePattern<mydialect::AddIOp>`

```c++
#include "mlir/IR/PatternMatch.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mydialect/MyOps.h.inc" // 包含 AddIOp 定义

using namespace mlir;

// 定义一个用于优化 mydialect.addi 的 Rewrite Pattern
struct RemoveAddZeroPattern : OpRewritePattern<mydialect::AddIOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(mydialect::AddIOp op,
                                PatternRewriter &rewriter) const override {
    Value lhs = op.getLhs();
    Value rhs = op.getRhs();

    // 判断是否为常量 0（左边或右边）
    auto isZeroConst = [](Value val) -> bool {
      if (auto cst = val.getDefiningOp<arith::ConstantOp>()) {
        if (auto attr = cst.getValue().dyn_cast<IntegerAttr>())
          return attr.getValue().isZero();
      }
      return false;
    };

    // 优化规则：如果 rhs 是 0，结果就是 lhs
    if (isZeroConst(rhs)) {
      rewriter.replaceOp(op, lhs);  // 替换整个 Op 为 lhs
      return success();
    }

    // 规则2：如果 lhs 是 0，结果是 rhs
    if (isZeroConst(lhs)) {
      rewriter.replaceOp(op, rhs);
      return success();
    }

    // 否则不优化
    return failure();
  }
};
```

#### 3.2.2 注册并应用 Pattern

```c++
#include "MyPass.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "mydialect/MyOps.h.inc"

namespace mlir {
namespace {

struct RemoveMyAddZeroPass
    : public PassWrapper<RemoveMyAddZeroPass, OperationPass<func::FuncOp>> {

  void runOnOperation() override {
    func::FuncOp func = getOperation();
    MLIRContext *ctx = &getContext();

    RewritePatternSet patterns(ctx);
    patterns.add<RemoveAddZeroPattern>(ctx); // <<< 注册 Pattern

    if (failed(applyPatternsAndFoldGreedily(func, std::move(patterns))))
      signalPassFailure();
  }
};

} // namespace

std::unique_ptr<Pass> createRemoveMyAddZeroPass() {
  return std::make_unique<RemoveMyAddZeroPass>();
}

} // namespace mlir

```

