**定义AddConst算子**

```c++
def Top_AddConstOp: Top_Op<"AddConst",
    [SupportFuseRelu, SupportPermuteMove, ScalarProducer, ScalarConsumer]> {
  let arguments = (
    ins 
    AnyTensor:$input,                     // 输入是一个张量
    F64Attr:$const_val,                   // 常数值
    DefaultValuedAttr<BoolAttr, "false">:$do_relu,
    DefaultValuedAttr<F64Attr, "-1.0">:$relu_limit,
    DefaultValuedAttr<BoolAttr, "false">:$is_scalar
  );
  let results = (outs AnyTensor:$output);
}
```

可以看到：

- 输入 (`input`)：是任意张量
- 常数 (`const_val`)：是一个 `F64Attr`（浮点数属性，不是 tensor）
- 输出 (`output`)：还是一个张量

**对比普通AddOp**

```c++
def Top_AddOp: Top_Op<"Add", [...]> {
  let arguments = (ins AnyTensor:$lhs, AnyTensor:$rhs);
  let results   = (outs AnyTensor:$output);
}
```

- `AddOp` 的两个输入都是 **张量**

- 而 `AddConstOp` 的第二个输入是 **一个常数属性**
