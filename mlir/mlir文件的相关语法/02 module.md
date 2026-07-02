

```mlir
#loc = loc(unknown)
module @yolov5s attributes {
    module.FLOPs = 16683511600 : i64, 
    module.chip = "ALL", 
    module.platform = "ONNX", 
    module.state = "TOP_F32", 
    module.top_run_mode = "STATIC", 
    module.weight_file = "yolov5s_top_f32_all_weight.npz"
} {
  func.func @main(%arg0: tensor<1x3x640x640xf32> loc(unknown)) -> 
    (tensor<1x3x80x80x85xf32>, tensor<1x3x40x40x85xf32>, tensor<1x3x20x20x85xf32>) {
    // 函数体内容...
  }
}
```

#### 1、module

**（1）模块属性分析**

- `module.FLOPs = 16683511600 : i64`: 模型的计算量（浮点运算次数）
- `module.chip = "ALL"`：目标芯片架构
- `module.platform = "ONNX"`：源框架是ONNX
- `module.state = "TOP_F32"`：处于全精度浮点32位状态
- `module.top_run_mode = "STATIC"`：静态执行模式
- `module.weight_file = "yolov5s_top_f32_all_weight.npz"`：权重文件

**符号(Symbol)**

```mlir
module @yolov5s { ... }      
func.func @main(...) { ... }  
```

- **全局可见**：在模块范围内唯一
- **可被引用**：其他操作可以通过符号名称引用它

**（2）`atrribute`

```mlir
module @name attributes { ... } { ... }
```

```mlir
"dialect.op"() {xxx = value} : () -> type
```

#### 2、函数分析

```mlir
func.func @main(%arg0: tensor<1x3x640x640xf32>) -> 
  (tensor<1x3x80x80x85xf32>, tensor<1x3x40x40x85xf32>, tensor<1x3x20x20x85xf32>)
```

这个虽然在mlir里是一个函数级的内容

- `dialect.op (operands) : (tensor) -> (tensor) `

**输入张量**：

- `tensor<1x3x640x640xf32>`：batch=1, channels=3, height=640, width=640

**三个输出张量**（对应YOLOv5的三个检测头）：

1. `tensor<1x3x80x80x85xf32>`：大特征图，检测小目标
	- 1: batch, 3: 每个位置的anchor数, 80x80: 网格大小, 85: 4(bbox)+1(conf)+80(class)
2. `tensor<1x3x40x40x85xf32>`：中等特征图
3. `tensor<1x3x20x20x85xf32>`：小特征图，检测大目标