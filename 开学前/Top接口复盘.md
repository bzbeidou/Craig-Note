## 1、复习

### 1.1 ShapeInferOpInterface

**用途**：推导算子输出的 shape（静态推导）。
 **适用算子**：元素级运算（Abs、Add）、卷积（Conv）、池化（Pool）等。

TableGen 声明

```
def Top_AbsOp : Top_Op<"Abs", [ShapeInferOpInterface]> {
	let summary = "Abs value op";
	let arguments = (ins F32Tensor:$input);
	let resluts = (outs F32Tensor:$input);
}
```

生成的 C++（部分）

```c++
class AbsOp : public Op<AbsOp, ShapeInterOpInterface::Trait>{
public:
    LogicalResult inferShape();
}
```

C++ 实现位置（`lib/Dialect/Interfaces/Abs.cpp`）

```c++
LogicalResult AbsOp::interShape(){
    getResult().setType(getInput().getType());
    return success();
}
```



### 1.2FoldInterface / Canonicalization

**用途**：常量折叠、Op 简化（比如 x + 0 → x）。
 **适用算子**：加法、乘法、ReLU 等有简化规则的运算。

**TableGen 声明**

```
def Top_AddOp : Top_Op<"Add", [FoldInterface]> {
	let summary = "Elementwise addition";
}
```

生成的 C++

```c++
class AddOp : public Op<AddOp, FoldInterface::Trait>{
public:
    OpFoldResult fold(ArrayRef<Attribute> operands);
}
```

C++ 实现（`lib/Dialect/Interfaces/Add.cpp`）

```c++
OpFolfResult AddOP::fold(ArrayRef<Attribute> operands){
    if(operands[0] && operands[1]){
    	auto lhs = operands[0].cast<FloatAttr>().getValueAsDouble();
    	auto rhs = operands[1].cast<FloatAttr>().getValueAsDouble();
    	return FloatAttr::get(getType(), lhs + rhs);      
    }
    return {};
}
```

### 1.3 VerifyOpInterface

**用途**：检查算子参数、shape 等是否合法。
 **适用算子**：输入必须满足特定条件的（Conv、MatMul 等）。

TableGen 声明

```
def Top_ConvOp : Top_Op<"Conv", [VerifyOpInterface]> { ... }
```

生成的 C++

```c++
class TopConv : public Op<ConvOp, VerifyOpInterface::Trait>{
public:
	LogicalResult verify();
}
```

C++ 实现（`lib/Dialect/Interfaces/Conv.cpp`）

```c++
LogicalResult TopConv::verify(){
  if (getKernelSize() <= 0)
    return emitOpError("kernel size must be positive");
  return success();
}
```

### 1.4 Lowering 接口

**用途**：将高层 Dialect（`top`）的算子降低到硬件 Dialect（`tpu`）或 LLVM Dialect。
 **适用算子**：几乎所有最终要跑在硬件上的算子。

TableGen 声明

```
def TopAbsOp : Top_Op<"Abs", [LoweringF32, LoweringINT8]>{
	...
}
```

生成的 C++

```c++
class TopAbs : public Op<AbsOp,LoweringF32::Trait, LoweringINT8::Trait >{
  void lowering_f32(PatternRewriter &rewriter);
  void lowering_int8(PatternRewriter &rewriter, bool asymmetric);
}
```

C++ 实现（`lib/Dialect/Interfaces/Abs.cpp`）

```c++
void AbsOp::lowering_f32(PatternRewriter &rewriter) {
  rewriter.replaceOpWithNewOp<tpu::AbsOp>(getOperation(), getType(), getInput());
}
```

### 1.5 BroadcastLikeInterface

 **用途**：提供统一的广播 shape 计算逻辑。
 **适用算子**：Add、Mul 等需要广播的元素级运算。

TableGen 声明

```
def Top_AddOp : Top_Op<"Add", [BroadcastLikeInterface]> { ... }
```

生成的 C++

```c++
class AddOp : public Op<AddOp, BroadcastLikeInterface::Trait> {
  Shape getBroadcastShape(Value lhs, Value rhs);
};
```

C++ 实现

```
Shape AddOp::getBroadcastShape(Value lhs, Value rhs) {
  // 计算两个输入的广播结果 shape
}
```



## 2 调用阶段

假设我们有一个 `AbsOp`，它在 TableGen 里声明了实现 `ShapeInferOpInterface`：

```
def Top_AbsOp : Top_Op<"Abs", [ShapeInferOpInterface]> {
  let arguments = (ins F32Tensor:$input);
  let results   = (outs F32Tensor:$output);
}

```

TableGen 会生成：

```
class AbsOp : public Op<AbsOp, ShapeInferOpInterface::Trait> {
public:
  LogicalResult inferShape();  // 声明（无实现）
};
```

然后我们在 `Abs.cpp` 里实现：

```c++
LogicalResult AbsOp::inferShape() {
  getResult().setType(getInput().getType()); // 输出 shape 和输入相同
  return success();
}
```

**Pass 调用接口**

```c++
struct ShapeInferencePass : public PassWrapper<ShapeInferencePass, OperationPass<ModuleOp>> {
  void runOnOperation() override {
    getOperation().walk([&](Operation *op) {
      // 检查当前 Op 是否实现了 ShapeInferOpInterface
      if (auto shapeOp = dyn_cast<ShapeInferOpInterface>(op)) {
        if (failed(shapeOp.inferShape())) {
          op->emitError("Shape inference failed");
          signalPassFailure();
        }
      }
    });
  }
};

```



## 3 Pass调用

Pass 里两种常见调用方式

| 调用方式         | 适用场景                             | TPU-MLIR 例子                  |
| ---------------- | ------------------------------------ | ------------------------------ |
| **直接调接口**   | 需要对所有实现该接口的算子做统一操作 | Shape 推导、Lowering、常量折叠 |
| **应用 Pattern** | 需要基于匹配规则改写 IR              | 算子融合、算子替换、规范化     |

**接口调用**

Pass 不关心算子名字，只要算子实现了接口，就可以调用它的方法。

```
op->walk([&](ShapeInferOpInterface shapeOp) {
    shapeOp.inferShape();
});
```

`ShapeInferOpInterface` → 统一推导 shape

`LoweringInterface` → 统一降到 tpu dialect

`FoldInterface` → 常量折叠

**Pattern 调用（匹配 + 重写）**

Pattern 是 `OpRewritePattern<OpType>` 的子类

融合 Add + Relu

```c++
struct FuseAddReluPattern : public OpRewritePattern<AddOp> {
  LogicalResult matchAndRewrite(AddOp op, PatternRewriter &rewriter) const override {
    if (!op.hasRelu()) return failure();
    auto fused = rewriter.create<FusedAddReluOp>(...);
    rewriter.replaceOp(op, fused.getResult());
    return success();
  }
};

```

在 Pass 里注册：

```
RewritePatternSet patterns(&getContext());
patterns.add<FuseAddReluPattern>(&getContext());
applyPatternsAndFoldGreedily(module, std::move(patterns));

```

典型用途：

- 算子融合
- 算子替换（比如替换成更底层的算子）
- 规范化 IR（canonicalization）



**TPU-MLIR 中两者的结合**

很多时候 **Pattern 里会调用接口方法**：

例子：

- 一个 `LoweringPattern` 可以先调用接口的 `inferShape()` 确定 shape
- 再根据接口的 lowering 实现创建新 Op

伪代码：

```c++
LogicalResult matchAndRewrite(AbsOp op, PatternRewriter &rewriter) const override {
  op.inferShape(); // 接口调用
  auto newOp = rewriter.create<tpu::AbsOp>(...);
  rewriter.replaceOp(op, newOp.getResult());
  return success();
}

```

这样：

- **Pattern** 决定“匹配哪些 Op 以及怎么改”
- **接口** 提供“算子该怎么干活”的具体实现

**总结**

- **接口**：像“工具箱”，统一定义功能入口；Pass 可批量调用它们
- **Pattern**：像“规则表”，匹配 IR 并应用改写；Pass 通过 `applyPatternsAndFoldGreedily()` 等方法执行
- 在 TPU-MLIR 里：
	- Shape 推导 Pass → 直接调接口
	- 常量折叠 Pass → 常用接口 + Pattern 混用
	- Lowering Pass → 通常是 Pattern 调用接口
