#### 1、 `RewritePatternSet` 

```c++
mlir::RewritePatternSet patterns(context);
```

#### 2、`patterns->add<...>()`

```c++
patterns->add<PatternA, PatternB, PatternC>(context, benefit);
```

#### 3、benifit

包装类

```
mlir::PatternBenefit benefit;
```

设某个 `Add` 恰好满足两个条件：
 既是多输入 Add，又后面接了 Relu。

那么问题来了：两个 pattern 都能重写它，那该选谁？

 MLIR 内部就会比较它们的 `PatternBenefit` 值。

- 并不是并行执行，而是在执行的时候会先判断执行哪一个再具体执行

**这里是 `applyPatternsAndFoldGreedily()` 的逻辑**

```c++
struct FuseAddReluPattern : public OpRewritePattern<top::AddOp> {
  FuseAddReluPattern(mlir::MLIRContext *ctx)
      : OpRewritePattern<top::AddOp>(ctx, /*benefit=*/3) {}

  LogicalResult matchAndRewrite(top::AddOp op, PatternRewriter &rewriter) const override {
    if (!hasReluUser(op)) return failure();
    // Fuse Add + Relu
    ...
    return success();
  }
};

struct ConvertMultiInputAdd : public OpRewritePattern<top::AddOp> {
  ConvertMultiInputAdd(mlir::MLIRContext *ctx)
      : OpRewritePattern<top::AddOp>(ctx, /*benefit=*/8) {}

  LogicalResult matchAndRewrite(top::AddOp op, PatternRewriter &rewriter) const override {
    if (op.getInputs().size() <= 2)
      return failure();
    ...
    return success();
  }
};

```

```c++
patterns->add<FuseAddReluPattern, ConvertMultiInputAdd>(ctx);

```

运行 `applyPatternsAndFoldGreedily()` 

#### 这里传入的benifit有什么用？

```c++
patterns->add<APattern, BPattern>(ctx, 5);
```

- APattern(ctx, 5)
- BPattern(ctx, 5)

```c++
  ConvertMultiInputAdd(mlir::MLIRContext *context, int benefit)
      : OpRewriterPatternEx<top::AddOp>(context, "ConvertMultiInputAdd") {}
```

#### add set

```c++
  patterns
      ->add<ConvertMultiInputAdd, ConcatToSwapDimInner, NoneZeroFixRowMajor>(
          patterns->getContext(), 8);
  patterns->add<patterns::SqueezeToReshapePattern>(patterns->getContext());
  patterns->add<patterns::UnsqueezeToReshapePattern>(patterns->getContext());
```

