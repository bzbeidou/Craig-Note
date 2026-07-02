###  阶段 1：图结构分析

- `ShapeInferPass  `  形状推理，确保每个 Op 的输入输出 shape 都正确
- `FusePreprocessPass`  将输入层的 normalize/resize 等融合进前置算子

目标：理解 IR 的结构变化（输入输出张量、Op 连接、图优化）。

------

###  阶段 2：芯片相关 Pass

- `ProcessorAssignPass`  将算子映射到对应的计算单元（如 TPU core）
- `ProcessorOptimizePass`  针对芯片结构优化，如指令复用/访存优化

目标：理解编译器如何分配算力与优化访存。

------

###  阶段 3：量化阶段

- `ImportCalibrationTablePass`  读取量化校准数据，用于 int8 推理
- `QDQConvertPass ` 插入 Quant/Dequant 算子，做 fake quant

 目标：理解 int8 推理如何通过插入 Q/DQ 节点实现。

------

### 阶段 4：后处理 + 收尾

-  `AddPostprocessPass`  添加后处理层，如 softmax、topk、reshape 等
- `DeinitPass`  清理所有临时信息，为 Codegen 做准备

 目标：理解图从中间 IR 到 codegen 输入的最终形态。

#### 量化阶段要在芯片相关Pass的后面？

量化要做两件事：

- 在 FP32 图中插入 `quant` / `dequant` 节点；
- 调整权重与激活值的 scale、zero-point。

```mlir
%1 = "top.Conv"(%input, %weight)
%2 = "top.BatchNorm"(%1)
```

```mlir
%1 = "top.ConvBNFused"(%input, %weight, %bn)
```

```mlir
%1 = "top.Conv"(...)
%1q = "top.Dequant"(%1)
```

##### 量化精度和layout与芯片相关

不同芯片对量化的要求不同，例如：比如BM1684量化精度是int8，然后layout要求是NCHW，要求对称量化。但是BM1684X量化精度是int16，layout要求是NHWC，支持非对称。那么量化要依赖这些信息，所以先要执行 ProcessorAssignPass确定这些信息
