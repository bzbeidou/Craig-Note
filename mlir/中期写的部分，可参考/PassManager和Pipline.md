### 1、PassManager

就是管理Pass的容器，需要一个PassManager然后去把Pass都放进来管理

```c++
mlir::PassManager pm(ctx);

// 程序里手动构建 pipeline
pm.addPass(mlir::createCanonicalizerPass()); // 规范化
pm.addPass(createMyCustomPass());             // 自定义 Pass
pm.addPass(mlir::createCSEPass());            // 公共子表达式消除

if (failed(pm.run(moduleOp))) {
  llvm::errs() << "Pass pipeline failed\n";
}

```

### 2、Pipeline  

调度顺序，因为后续通过PassManager调用这些Pass的时候是有顺序的

关键

**先后依赖等等**：有些 pass 会制造出后续 pass 能处理的形态（例如先 `canonicalize` 把 `add(x,0)` 变掉，后面的 pass 才能看到更简洁的图并做融合）。

- 在 pipeline 里把 **轻量且高回报（cheap & effective）** 的 pass 放前面。
- 把 **昂贵的 pass** 放在确认宽松的简化后再运行（以避免浪费）。



**实际上执行的时候就是流水线Pipeline的感觉，如下：**

当调用 `pm.run(moduleOp)`：

1. PassManager 遍历内部的 Pass 列表（pipeline）。
2. 按顺序调用每个 pass 的 `runOnOperation()`。
3. 每个 pass 对 IR，也就是这个module的内容做改写或分析。

### 3、 PassRegistry和PassRegistrition

这个挺难理解



