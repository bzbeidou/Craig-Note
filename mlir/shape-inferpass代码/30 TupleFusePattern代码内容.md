

元组融合

```c++
class TupleFusePattern : public OpRewriterPatternEx<TupleOp> {
public:
  TupleFusePattern(mlir::MLIRContext *context)
      : OpRewriterPatternEx<TupleOp>(context, "TupleFusePattern") {}

  LogicalResult matchAndRewriteImpl(TupleOp op,
                                    PatternRewriter &rewriter) const override {
    auto out = op.getOutput();
    for (auto user : op->getUsers()) {
      std::vector<Value> operands;
      for (auto opd : user->getOperands()) {
        if (opd == out) {
          for (auto v : op.getOperands()) {
            operands.push_back(v);
          }
        } else {
          operands.push_back(opd);
        }
      }
      user->setOperands(operands);
    }
    rewriter.eraseOp(op);
    return success();
  }
  bool shouldPrint(TupleOp op) const override { return false; }
};
```

```mlir
%tuple = "top.Tuple"(%x, %y)
%r = "top.Concat"(%tuple, %z)
```

因为要去冗余结构，Tuple为了方便某些导入流程（比如ONNX->MLIR）,

#### 针对TupleOp的一个算子

- 输入是三个SSA值a%, b%, c%，然后输出就是将他们打包成一个tuple了，就是打包了

- 返回值这个SSA值%tuple当前也可以被其他算子使用

**但是在一些场景之下，后续算子希望直接拿到 `%a, %b, %c` 这几个 tensor，而不是打包后的 tuple。**

于是，这个 `TupleFusePattern` 的作用就是：**把 `Tuple` 展开（fuse 展开）成原始输入，把下游算子的输入扁平化。**

#### 核心代码逻辑详解

让我们逐行来看：

```
LogicalResult matchAndRewriteImpl(TupleOp op,
                                  PatternRewriter &rewriter) const override {
```

- `TupleOp op`：匹配到的 Tuple 操作；
- `PatternRewriter`：用于修改 IR 的工具类。

取出 Tuple 的输出

```
auto out = op.getOutput();
```

- 每个算子（Op）可能有多个输出；
- 但 `TupleOp` 一般只有 **一个 tuple 类型输出**，里面包含多个元素。

比如：

```
%tuple = "top.Tuple"(%x, %y, %z)
```

这里的 `out` 就是 `%tuple`。

遍历所有使用这个 tuple 的算子（users）

```
for (auto user : op->getUsers()) {
```

`getUsers()` 返回所有使用 `%tuple` 这个 Value 的 Operation。
 比如：

```
%tuple = "top.Tuple"(%x, %y)
"top.Concat"(%tuple)
"top.Print"(%tuple)
```

那 users 就有两个：`top.Concat` 和 `top.Print`。

对每个使用者的输入进行检查和替换

```
std::vector<Value> operands;
for (auto opd : user->getOperands()) {
```

- `user->getOperands()` 拿到这个使用者的所有输入；
- 然后挨个判断是否是 `TupleOp` 的输出。
-  如果输入是 tuple 的输出，展开 tuple 的原始输入

```
if (opd == out) {
  for (auto v : op.getOperands()) {
    operands.push_back(v);
  }
} else {
  operands.push_back(opd);
}
```

这几行是关键。

假设原来有：

```
%tuple = "top.Tuple"(%a, %b)
%r = "top.Concat"(%tuple, %c)
```

那么这里：

- `op.getOperands()` = `%a, %b`
- `user->getOperands()` = `%tuple, %c`

于是替换逻辑执行后：

```
%r = "top.Concat"(%a, %b, %c)
```

把 tuple 打包的内容直接展开传给使用者。

用新的操作数覆盖用户算子的输入

```
user->setOperands(operands);
```

这行调用 MLIR 提供的通用接口，
 表示：

> 替换掉旧的输入（operands）列表，用我们新构建的 `operands`。

删除原来的 Tuple 节点

```
rewriter.eraseOp(op);
return success();
```

- 因为它的输出已经不再被使用；
- 所以直接删除，图结构变得更干净。