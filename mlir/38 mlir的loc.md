```mlir
#loc = loc(unknown)
module @yolov5s attributes {module.FLOPs = 16683511600 : i64, module.chip = "ALL", module.platform = "ONNX", module.state = "TOP_F32", module.top_run_mode = "STATIC", module.weight_file = "yolov5s_top_f32_all_weight.npz"} {
  func.func @main(%arg0: tensor<1x3x640x640xf32> loc(unknown)) -> (tensor<1x3x80x80x85xf32>, tensor<1x3x40x40x85xf32>, tensor<1x3x20x20x85xf32>) {
    %0 = "top.Input"(%arg0) {channel_format = "nchw", do_preprocess = true, keep_aspect_ratio = true, keep_ratio_mode = "letterbox", mean = [0.000000e+00, 0.000000e+00, 0.000000e+00], pad_type = "center", pad_value = 0 : i64, pixel_format = "rgb", resize_dims = [640, 640], scale = [0.0039216000586748123, 0.0039216000586748123, 0.0039216000586748123]} : (tensor<1x3x640x640xf32>) -> tensor<1x3x640x640xf32> loc(#loc1)
    %1 = "top.Weight"() : () -> tensor<32x3x6x6xf32> loc(#loc2)
    
   ...
   
#loc1 = loc("images")
#loc2 = loc("model.0.conv.weight")
#loc3 = loc("model.0.conv.bias")
...
```

#### 1 字符串

假设一开始有一个 **ONNX 模型** 或 **PyTorch 模型**：

```
Conv_1 (name="conv1")
 ↓
ReLU_2 (name="relu1")
 ↓
Output
```

```onnx
node {
  op_type: "Conv"
  name: "conv1"
  input: "input_0"
  output: "conv1_out"
}
```

```mlir
%1 = "tpu.Conv"(%0) {kernel = [3, 3]} :
      (tensor<1x3x224x224xf32>) -> tensor<1x64x112x112xf32>
      loc("conv1")
```

#### 2 tpumlir代码

```mlir
%1 = "tpu.Conv"(%input, %weight) : (...) -> (...) loc("conv1")
```

```mlir
%1 = "tpu.Conv"(%input, %weight) : (...) -> (...) loc(#loc1)
#loc1 = loc("conv1")
```

```mlir
#loc_conv1 = loc("conv1")

%1 = "top.Conv"(%x, %w) : (...) -> (...) loc(#loc_conv1)
%2 = "top.BatchNorm"(%1) : (...) -> (...) loc(#loc_conv1)
%3 = "top.Relu"(%2) : (...) -> (...) loc(#loc_conv1)
```



```c++
%1 = "top.Conv"(%x, %w) : (...) -> (...) loc("conv1")
%2 = "top.BatchNorm"(%1) : (...) -> (...) loc("conv1")
%3 = "top.Relu"(%2) : (...) -> (...) loc("conv1")

```

#### 3 从解析.mlir

**（1）例子**

```mlir
func.func @main(%arg0: tensor<1x3x224x224xf32> loc("input")) -> tensor<1x3x224x224xf32> {
  %0 = "top.Conv"(%arg0) : (tensor<1x3x224x224xf32>) -> tensor<1x3x224x224xf32> loc("conv1")
  return %0 : tensor<1x3x224x224xf32>
}
```

两个loc

**（2）TPUMLIR解析**

**所有的loc()也会被解析为某个Location**

**（3）Location对象  用的时候主要是用具体的子类对象**

**（4）c++层面的构造方法**

```c++
OpBuilder builder(context);
auto loc = NameLoc::get(builder.getStringAttr("conv1"));
```

这就创建了一个NameLoc

`NameLoc::get()` 在当前 `MLIRContext` 的 `AttributeUniquer` 表中查找是否已有相同 `StringAttr("conv1")`，如果有，则复用同一内存；否则创建新的实例。

**b 使用loc构造Operation**

```c++
auto convOp = builder.create<top::ConvOp>(
    loc, 
    resultType, input, weight);

```

在.mlir层面创建了如下内容

```mlir
%0 = "top.Conv"(%input, %weight) : (...) -> (...) loc("conv1")
```

Op底层的Operation，然后直接调用它的getLoc,然后后面的打印我们很熟悉了

```c++
Operation *op = ...;
Location loc = op->getLoc();  

if (auto nameLoc = loc.dyn_cast<NameLoc>()) {
  llvm::outs() << "Op name: " << nameLoc.getName() << "\n";
}

```
