#### 基本格式

```mlir
// 基本格式： %ssa_name = "dialect.op_name"(operands) {attributes} : (operand_types) -> result_types
%result = "tpu.Conv"(%input, %weight, %bias) { 
    kernel_shape = [3, 3], 
    strides = [1, 1], 
    pads = [0, 0, 0, 0] 
} : (tensor<1x3x224x224xf32>, tensor<32x3x3x3xf32>, tensor<1x32x1x1xf32>) -> tensor<1x32x224x224xf32>
```

- **结果**：`%result` 是操作产生的SSA值
- **操作名称**：通常是方言限定的。
- **操作数**：是提供给操作的输入SSA值
- **属性**：是编译时常量，影响操作的行为
- **类型**：操作类型和结果的类型。