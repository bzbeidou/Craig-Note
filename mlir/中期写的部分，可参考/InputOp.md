#### 1 InputOp

InputOp代表外部输入接口

```mlir
%0 = "top.Input"() {
  do_preprocess = true,
  resize_dims = [640, 640],
  mean = [0.0, 0.0, 0.0],
  scale = [0.0039216, 0.0039216, 0.0039216]
} : () -> tensor<1x3x640x640xf32>
```

#### 2 有输入的情况

```mlir
%0 = "top.Input"(%arg0) : (tensor<1x3x640x640xf32>) -> tensor<1x3x640x640xf32>
```



#### 3 一般只处理InputOp就行

在后续的 Pass 或 Lowering 流程中：

- 会被认为是 **模型计算的起点**；
- 下游算子（如 Conv、Add）会消费它；
- 它的类型（`tensor<...xf32>` 或带量化信息）决定整个模型的输入格式。



#### 4 直接传参数到实际计算算子不行吗？

**可以在MLIR中保存更多的相关属性**

这些信息无法存放在函数参数上（`%arg0` 只是类型信息），