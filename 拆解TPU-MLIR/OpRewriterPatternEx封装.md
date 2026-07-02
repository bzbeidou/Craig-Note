就是继承 `mlir::OpRewritePattern<OpType>`，然后写一些统一的代码，做了一个封装wrapper

**减少重复代码**

- 如果你写 100 个 pattern，每个都要写一遍 “统计 + 打印日志” → 代码很冗余。
- 封装到基类里，子类专注于核心逻辑。

**统一管理**

- 日志风格、计数逻辑都由基类控制，避免每个 pattern 写法不一致。

**调试和分析方便**

- 通过计数器和日志，你能快速知道某个 Pass 中哪些 pattern 生效了，生效了多少次。

正常情况下，写一个 Pattern 是这样的：

```c++
class MyPattern : public mlir::OpRewritePattern<MyOp> {
public:
    using OpRewritePattern::OpRewritePattern;
    
    LogicalResult matchAndRewrite(MyOp op, PatternRewriter &rewriter) const override{
        return successs();
    }
}
```

 `OpRewriterPatternEx` 

 `matchAndRewrite`

``matchAndRewriteImpl`

```c++
LogicalResult matchAndRewrite(SourceOp op, PatternRewriter &rewriter) const override {
  LogicalResult result = matchAndRewriteImpl(op, rewriter); // 调子类的核心逻辑
  if (succeeded(result)) {
    // 额外功能：计数 & 日志
  }
  return result;
}
```

```c++
class MyPatternEx : public OpRewriterPatternEx<MyOp> {
public:
  using OpRewriterPatternEx::OpRewriterPatternEx;

protected:
  LogicalResult matchAndRewriteImpl(MyOp op, PatternRewriter &rewriter) const override {
    // 只写核心重写逻辑
    return success();
  }
};
```




