# TopToTpuPass runOnOperation

## 1、初始化逻辑

**源代码：**

```c++
  module_ = getOperation();
  ctx_ = &getContext();
  mainFunc_ = module::getMainFuncOp(module_);
  LoweringConfig::isQuantized = false;
  module::setGroupQuantInfo(quantGroupSize, quantSymmetric);
  if (weightFileName != "") {
    module::setWeightFileName(weightFileName);
  }
  int64_t flops = module::getFLOPs();

```

首先需要获取当前执行的顶层操作，确定pass 被应用在哪个模块上的操作



```mlir
module {
  func.func @main() {
    ...
  }
}
```



这里整个 `module {}` 就是 `getOperation()` 返回的对象，也就是`mlir::ModuleOp module_;`



然后获取 MLIR 的全局 `MLIRContext` 上下文对象，从 module 中找到主函数 `main`，类型是 `FuncOp`。

mlir中的典型主函数：



```mlir
func.func @main(%arg0: tensor<1x3x224x224xf32>) -> () {
  ...
}
```



然后设置一个是否量化的全局标志， `LoweringConfig::isQuantized = false;`把量化的分组信息（Group Size 和是否 symmetric）写入 module 的属性中。后续如果有其它 Pass 需要知道量化设置，可以通过属性来读取这些信息，而不用全局变量。

会把类似下面的属性加到 `module` 上：



```mlir
module attributes {quant.group_size = 64, quant.symmetric = true} {
  ...
}
```



如果设置了权重文件名（比如从命令行参数传入），就把它记录在 module 属性中，供后续保存模型时使用。



**源码：**

```c++
int64_t flops = module::getFLOPs();
if (flops == 0) {
  mainFunc_.walk([&](FlopsInterface op) {
    flops += op.getFLOPs();
  });
  module::setFLOPs(flops);
}

```

如果还没有统计 FLOPs遍历一次主函数中的 Op，**自动计算 FLOPs 总量**并写入 `Module` 属性中。

通过 `getFLOPs()` **读取 module 上是否存在某个属性（如 flops 值）**。比如module IR可能有以下属性，此时则会返回12345678：

```mlir
module attributes { flops = 12345678 } {
  ...
}
```



如果当前没有统计过 FLOPs（默认为 0），就开始进行 FLOPs 的自动统计，因为有的模型可能在构建时就已经加过flops属性了。

通过 `mainFunc_.walk(...)` 遍历主函数中的所有操作（Op）；某个 Op 实现了 `FlopsInterface` 接口，就调用其 `getFLOPs()` 方法，累加到 `flops` 中，把刚刚统计出来的总 FLOPs 写回 Module 的属性中。



## 2、转换逻辑一



```c++
  RewritePatternSet patterns(ctx_);
  patterns.clear();
  patterns.add<TryInsertTileBinaryPattern<top::SubOp>,
               TryInsertTileBinaryPattern<top::MaxOp>,
               TryInsertTileBinaryPattern<top::MinOp>,
               TryInsertTileBinaryPattern<top::CompareOp>,
               TryInsertTileMatMulPattern>(ctx_);
  if (!module::isBM1684XFamily()) {
    patterns.add<TryInsertTileBinaryPattern<top::AddOp>,
                 TryInsertTileBinaryPattern<top::MulOp>>(ctx_);
  }
  applyPatternsAndFoldGreedily(module_, std::move(patterns));
  patterns.clear();
```

这部分就是MLIR中的Pass的知识，往 `patterns` 中添加多个 Pattern 类，

```
%1 = top.sub %a, %b
```

转换为：

```
%a_tiled = top.tile %a
%b_tiled = top.tile %b
%1 = top.sub %a_tiled, %b_tiled
```



## 3、转换逻辑二



| Tensor Name | Scale | Zero Point | Type |
| ----------- | ----- | ---------- | ---- |
| conv1_out   | 0.12  | 128        | int8 |
| input_0     | 0.007 | 0          | int8 |

- **判断当前模型是否已经量化**

检查当前模型的状态state是不是 `TOP_QUANTIZED`，判断模型是否在更早阶段就已经做了静态量化处理。

**补充模型状态

在模型已经量化的情况下，设置为非对称量化（asymmetric=true），默认采用非对称模式；

在模型未量化的情况下，明确当前模型还是浮点状态，然后设置非对称量化的标志位，并执行校准流程 calibration。

- **为特定硬件执行量化逻辑**

总体结构大致划分：

```c++
if ((isBM1684X || isBM1688) && 模型未量化 && 模式为 INT8/UINT8) {
  1. MatMul 设置 per-channel 属性；
  2. MatMul + Add 设置 output_int16 属性；
  3. 检测 YOLO 模型结构并设置 int16；
}
```

**给`top::MatMulOp`设置per-channel属性**

per-channel quantization针对每个通道（channel）设置不同的 scale，适合 MatMul 和 Conv，提高精度，这个属性在 lowering 阶段被读取，从而改变生成的 MLIR/TPU 指令结构。

**给 MatMul 设置 int16 输出条件**

有些硬件不支持 float 中间结果，需要明确指示中间输出使用 int16，如果 MatMul 后面接的是 Add（加 bias），并且目标是 F16/BF16（量化的一种形式），则把中间输出设置为 int16；

**检测是否是 YOLO 结构，并设置 int16 输出**（待补充）

**针对MARS3和SGTPUV8的内容**

- **形状转换**

根据芯片型号（BM1684X 或 BM1684）来注册不同的 shape 类算子的转换规则，从 `top.shape` 等 op 转为 `tpu.shape` 相关 op

内部依然是通过 `.add<...>(ctx_)` 添加进 `RewritePatternSet patterns` 的。

**shape的转换为什么要单独拿出来？**

## 4、转换逻辑四

这里是将 Top Dialect 中的大部分算子（Op）转换为 Tpu Dialect 中的对应实现

```c++
if (module::isBM1684XFamily() || module::isBM1690Family()) { 
    bm1684x::populateTopToTpuConversionPatterns(&patterns);
  } else if (module::isBM1684Family()) {
    bm1684::populateTopToTpuConversionPatterns(&patterns);
  } else if (module::isCV18xx()) {
    cv18xx::populateTopToTpuConversionPatterns(&patterns);
  } else {
    // 如果芯片类型不匹配，则直接报错
    llvm_unreachable("Not Implemented");
  }
  // 设置 GreedyRewrite 配置，只执行一次重写
  auto config = GreedyRewriteConfig();

  config.maxIterations = 1; // apply each pattern only once.

  applyPatternsAndFoldGreedily(module_, std::move(patterns), config);
  // adjust reshape
  patterns.clear();
```

最后通过`module::updateModuleTypes()`重新整理并同步 Module 中所有 Operation 的类型信息，确保类型一致性。

```c++
bool hasTopOp = false;
mainFunc_.walk([&](Operation *op) {
  if (isa<top::WeightOp, top::NoneOp, top::InputOp, ModuleOp, FuncOp, ReturnOp>(op)) {
    return;
  }
  if (!isa<tpu::TpuDialect>(op->getDialect())) {
    op->dump();
    hasTopOp = true;
  }
});
```

