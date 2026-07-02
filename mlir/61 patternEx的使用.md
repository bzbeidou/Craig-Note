

```c++
template <typename SourceOp>
class OpRewriterPatternEx : public mlir::OpRewritePattern<SourceOp> {
public:
  OpRewriterPatternEx(mlir::MLIRContext *context,
                      llvm::StringRef patternName = "",
                      mlir::PatternBenefit benefit = 1)
      : mlir::OpRewritePattern<SourceOp>(context, benefit),
        patternName(patternName) {}

  mlir::LogicalResult
  matchAndRewrite(SourceOp op, mlir::PatternRewriter &rewriter) const override {
    mlir::LogicalResult result = matchAndRewriteImpl(op, rewriter);
    if (mlir::succeeded(result)) {
      if (!patternName.empty()) {
        std::lock_guard<std::mutex> lock(
            tpu_mlir::module::patternMatchCountsMutex);
        ++tpu_mlir::module::patternMatchCounts[patternName]; //pattern执行了多少次
      }
      if (shouldPrint(op) && !patternName.empty()) {
        //  #todo : print opname,no save has bug need to solve,this is a
        //  temporary solution
        PASS_LOG_DEBUG_BLOCK({
          llvm::outs() << patternName << " : " << op.getOperationName()
                       << " succeed!";
        });
      }
    }
    return result;
  }

protected:
  virtual mlir::LogicalResult
  matchAndRewriteImpl(SourceOp op, mlir::PatternRewriter &rewriter) const = 0;

  virtual bool shouldPrint(SourceOp op) const { return true; }

private:
  std::string patternName;
  static void printPatternMatchCounts() {
    std::lock_guard<std::mutex> lock(tpu_mlir::module::patternMatchCountsMutex);
    for (const auto &entry : tpu_mlir::module::patternMatchCounts) {
      std::cout << "Pattern [" << entry.first << "] matched " << entry.second
                << " times.\n";
    }
  }
};
```

