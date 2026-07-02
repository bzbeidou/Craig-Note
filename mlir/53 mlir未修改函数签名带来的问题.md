在我复现fusePresprocess的过程中执行完成pass之后提示了错误导致输出.mlir文件不能生成，也就是**函数签名未修改错误**

**mlir的语法还是相当严格的，生成会进行检查，然后检查出来函数签名未修改**



```mlir
func @main(%arg0: tensor<1x3x640x640xf32>) -> tensor<1x3x640x640xf32> {
  // 函数体，处理 %arg0
  %1 = "top.Input"(%arg0) {...} : tensor<1x3x640x640xf32> -> tensor<1x3x640x640xf32> loc(#loc1)
  ...
}
```

```mlir
func @main(%arg0: tensor<1x3x640x640xf32>) -> tensor<1x3x640x640xf32> {
  // 函数体，处理 %arg0
  %1 = "top.Input"(%arg0) {...} : tensor<1x3x640x640xui8> -> tensor<1x3x640x640xui8> loc(#loc1)
  ...
}
```



上面我们修改了inputOp的内容，自然要修改函数签名保持一致

至此问题解决了！！！