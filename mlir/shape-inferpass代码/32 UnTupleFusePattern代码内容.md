#### 

```c++
class UnTupleFusePattern : public OpRewriterPatternEx<UnTupleOp> {
public:
  UnTupleFusePattern(mlir::MLIRContext *context)
      : OpRewriterPatternEx<UnTupleOp>(context, "UnTupleFusePattern") {}

  LogicalResult matchAndRewriteImpl(UnTupleOp op,
                                    PatternRewriter &rewriter) const override {
    auto outs = op.getOutputs();
    auto ins = op.getInputs();
    if (outs.size() != ins.size()) {
      return failure();
    }
    for (auto it : llvm::zip(ins, outs)) {
      auto in = std::get<0>(it);
      auto out = std::get<1>(it);
      auto loc = module::getLoc(out);
      out.replaceAllUsesWith(in);
      module::setLoc(in, loc);
    }
    rewriter.eraseOp(op);
    return success();
  }
  bool shouldPrint(UnTupleOp op) const override { return false; }
};
```

#### 直接分析代码

**（1）获取输入输出**

```c++
auto outs = op.getOutputs();
auto ins = op.getInputs();
```

意思是：

- `UnTupleOp` 有若干个输入（`inputs`）,但是对于UnTupleOp一般只有一个输入
- 也有若干个输出（`outputs`）

```c++
%1:2 = "top.Tuple"(%a, %b) : (tensor<...>, tensor<...>) -> (tuple<tensor<...>, tensor<...>>)
%2, %3 = "top.UnTuple"(%1) : (tuple<...>) -> (tensor<...>, tensor<...>)
```

这里 `UnTuple` 有 1 个输入（tuple 类型），有 2 个输出（单独张量）。

**（2）检查输入输出数量匹配**

```cpp
if (outs.size() != ins.size()) {
  return failure();
}
```

检查输入输出个数是否对应。
 如果不一样（例如 tuple 中 2 个元素但输出 3 个），说明这个算子结构异常，不做优化。

**（3）替换使用关系**

```c++
for (auto it : llvm::zip(ins, outs)) {
  auto in = std::get<0>(it);
  auto out = std::get<1>(it);
  auto loc = module::getLoc(out);
  out.replaceAllUsesWith(in);
  module::setLoc(in, loc);
}
```

| 操作                         | 含义                                                |
| ---------------------------- | --------------------------------------------------- |
| `llvm::zip(ins, outs)`       | 把输入输出“配对”遍历，例如 `(in0,out0), (in1,out1)` |
| `out.replaceAllUsesWith(in)` | 把所有用到 `out` 的地方改成用 `in`                  |
| `module::setLoc(in, loc)`    | 保留原先 `out` 的位置信息（调试信息/源码映射）      |

**（4）删除算子**

**（5）返回成功**

```c++
return success();
```

告诉 MLIR 这个 pattern 匹配成功并完成了重写。
