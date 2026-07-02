### 1、运作逻辑

**（1）运作对象**



```
Pass<"...", ModuleOp>
```

- Module

**（2）作用**

````
 输入 `.mlir`：

```
%0 = top.Conv(%arg0, %w, %b) : (tensor<1x3x224x224xf32>, tensor<64x3x7x7xf32>, tensor<64xf32>) -> tensor<*xf32>
```

经过 ShapeInfer 后：

```
%0 = top.Conv(%arg0, %w, %b) : (tensor<1x3x224x224xf32>, tensor<64x3x7x7xf32>, tensor<64xf32>) -> tensor<1x64x112x112xf32>
```
 这里就是 **填充了推理出来的 shape**。
````

**（3）整个流程**

比如 TPU-MLIR 的 workflow：

```
onnx-mlir → (Init) → (ProcessorAssign) → (ShapeInfer) → (ExtraOptimize) → (Lowering)
```



### 2、TPU-MLIR的Pass介绍

1. **Init**

- **作用**：初始化整个模型 Module，比如设置基本参数（chip 频率、日志等级、权重存储位置等）。

2. **Deinit**

- **作用**：清理资源（可能是保存权重、释放中间状态等）。

3. **ProcessorAssign**

- **作用**：分配具体的硬件配置（芯片类型、量化模式、设备/核心数等）。

4. **ProcessorOptimize**

- **作用**：针对特定芯片做额外优化（在 Lowering 之前的阶段）。

	**还有很多，边学边看**
