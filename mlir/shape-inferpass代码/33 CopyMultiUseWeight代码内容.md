```c++
class CopyMultiUseWeight : public OpRewriterPatternEx<WeightOp> {
public:
  CopyMultiUseWeight(mlir::MLIRContext *context)
      : OpRewriterPatternEx<WeightOp>(context, "CopyMultiUseWeight") {}

  int getOperandIndex(Operation *op, Value operand) const {
    int n = op->getNumOperands();
    for (int i = 0; i < n; i++) {
      if (operand == op->getOperand(i)) {
        return i;
      }
    }
    llvm_unreachable("operand not found");
    return -1;
  }

  LogicalResult matchAndRewriteImpl(WeightOp op,
                                    PatternRewriter &rewriter) const override {
    std::vector<Operation *> users(op->user_begin(), op->user_end());
    if (users.size() <= 1) {
      return failure();
    }
    int idx = 0;
    for (auto user : users) {
      int operand_index = getOperandIndex(user, op.getOutput());
      auto new_weight = op.clone(std::to_string(idx));
      user->setOperand(operand_index, new_weight);
      idx++;
    }
    rewriter.eraseOp(op);
    return success();
  }
  bool shouldPrint(WeightOp op) const override { return false; }
};
```



#### 代码逐行解析

**（1）构造函数**

```c++
CopyMultiUseWeight(mlir::MLIRContext *context)
    : OpRewriterPatternEx<WeightOp>(context, "CopyMultiUseWeight") {}
```

- 这个 pattern 专门匹配 `WeightOp`
- 名字叫 `"CopyMultiUseWeight"`

**（2）辅助函数：获取操作数索引**

```cpp
int getOperandIndex(Operation *op, Value operand) const {
  int n = op->getNumOperands();
  for (int i = 0; i < n; i++) {
    if (operand == op->getOperand(i)) {
      return i;
    }
  }
  llvm_unreachable("operand not found");
  return -1;
}
```

这个函数就是查找“某个 operand 在 op 里的第几个位置”。

```mlir
%z = "top.Add"(%x, %y)
```

如果 operand 是 `%x`，返回 `0`；
 如果 operand 是 `%y`，返回 `1`。

这个索引后面用于 `setOperand()` 时替换权重。



**（3）匹配和重写主逻辑**

```c++
LogicalResult matchAndRewriteImpl(WeightOp op,
                                  PatternRewriter &rewriter) const override {
  std::vector<Operation *> users(op->user_begin(), op->user_end());
  if (users.size() <= 1) {
    return failure();
  }
```

解释：

- 获取当前 `WeightOp` 的所有使用者（即谁用了它的输出）。
- 如果使用者数量 ≤ 1（没被复用），不需要改写，直接 `failure()`。

**（4）针对每个用户复制权重**

```cpp
int idx = 0;
for (auto user : users) {
  int operand_index = getOperandIndex(user, op.getOutput());
  auto new_weight = op.clone(std::to_string(idx));
  user->setOperand(operand_index, new_weight);
  idx++;
}
```

**（5）最后删除原始`WeightOp`并不打印日志**