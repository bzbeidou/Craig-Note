# MLIR

## 1 、什么是`FlopsInterface`

这是MLIR中的一种Interface，就是接口机制

**定义方式：**

```tb
def FlopsInterface : OpInterface<"FlopsInterface"> {
  let methods = [
    InterfaceMethod<"int64_t", "getFLOPs", (ins), "返回当前Op的 FLOPs">,
  ];
}
```

如果一个Op实现了它，那么他就会提供自己的`getFlops()`实现逻辑

```c++
int64_t flops = module::getFLOPs();
if (flops == 0) {
  mainFunc_.walk([&](FlopsInterface op) {
    flops += op.getFLOPs();
  });
  module::setFLOPs(flops);
}
```



## 2、补充校准

校准是量化中的重要一步，收集激活值的范围信息（min/max），为后续生成量化 scale/zero-point 做准备。



# Top层

## 1、Top Dialect

```tb
def Top_Dialect : Dialect {
  let name = "top";
  let summary = "A topdialect for the TPU_MLIR specification";
  let cppNamespace = "::tpu_mlir::top";
}
```

| 字段名                             | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| `name = "top"`                     | 指定 Dialect 在 MLIR 中的名字是 `"top"`，在 `.mlir` 文件中看到的 `top.Conv`、`top.Relu` 就是这个名字 |
| `summary`                          | 只是说明文字，描述这个 Dialect 的用途，不参与实际生成        |
| `cppNamespace = "::tpu_mlir::top"` | 指定生成的 C++ 类应该放在哪个命名空间下                      |



```c++
namespace tpu_mlir {
namespace top {

class TopDialect : public ::mlir::Dialect {
  explicit TopDialect(::mlir::MLIRContext *context);

  void initialize();
  friend class ::mlir::MLIRContext;
public:
  ~TopDialect() override;
  static constexpr ::llvm::StringLiteral getDialectNamespace() {
    return ::llvm::StringLiteral("top");
  }
};

} // namespace top
} // namespace tpu_mlir

MLIR_DECLARE_EXPLICIT_TYPE_ID(::tpu_mlir::top::TopDialect)

```

`TopDialect` 负责注册所有 Top Dialect 的 Ops、Types、Attributes 等内容。

- 构造函数TopDialect(context)会在 `MLIRContext` 中构造一个该 Dialect 的实例，是自动注册机制的入口。

-  `initialize()` 注册 Op 和类型，在 `.td` 文件里写的所有 Op 和 Type，生成 C++ 后都会在这里进行注册。

```c++
void TopDialect::initialize() {
  addOperations<
    // 会列出所有从 TableGen 生成的 top.xxx Op
    ConvOp, ReluOp, MatMulOp, ...
  >();
}
```

-  `getDialectNamespace()`：返回 Dialect 的字符串名："top"

-  `MLIR_DECLARE_EXPLICIT_TYPE_ID(...)`宏是 MLIR 的一个类型注册机

## 2、Op

```
class Top_BaseOp<string mnemonic, list<Trait> traits = []> :
    Op<Top_Dialect, mnemonic, traits> ;
```

**定义了一个用于生成所有 Top Dialect 下 Op 的通用模板**，名字叫 `Top_BaseOp`，继承了 MLIR 提供的 `Op<>` 模板。

```
def Top_NoneOp : Top_BaseOp<"None"> {
  let summary = "none operator";

  let description = [{
    A none Op to return a NoneType.
  }];
  let results = (outs NoneType);
}
```

 `top.None` 的具体 Op，返回 `NoneType` 类型。没有输入，没有属性，输出类型是 `NoneType`

**WeightOp**

模型权重数据载入和管理

| 属性名           | 类型                           | 用途                                               |
| ---------------- | ------------------------------ | -------------------------------------------------- |
| `scale`          | `F64ArrayAttr`                 | 权重量化的 scale 参数（通常是 per-channel）        |
| `store_mode`     | 自定义枚举类型 `StoreModeAttr` | 权重存储方式，例如 float / int8 / bf16 等          |
| `allow_split`    | `I64ArrayAttr`                 | 用于标记某个维度是否允许 split                     |
| `do_compress`    | `BoolAttr`                     | 是否对权重做压缩存储（比如 nnvlc）                 |
| `bias0`, `bias1` | `I64Attr`                      | 自定义偏移，可能是压缩参数                         |
| `is_signed`      | `BoolAttr`                     | 标记权重是否为有符号数（比如 int8）                |
| `zero_guard`     | `BoolAttr`                     | 是否在0值上加“防护”，用于精度控制                  |
| `inline_bytes`   | `StrAttr`                      | 如果权重是内嵌的，则这里是 base64 编码后的字节数据 |

- Op的输出结果

```
let results = (outs AnyRankedTensor:$output);
```

会输出一个常规的张量。

- 优化规则  该 Op 有一个对应的 **Canonicalization Pattern**，也就是一个优化简化规则，可以找到对应的 `CanonicalizePattern` 注册和实现。。

- 为自动生成的C++ 类注入额外方法

```td
let extraClassDeclaration = [{
  std::shared_ptr<std::vector<T>> read();
  std::shared_ptr<std::vector<float>> read_as_float();
  ...
  static mlir::Value create(...);
  mlir::Value split(...);
}];

```

| 函数                                                     | 作用                                           |
| -------------------------------------------------------- | ---------------------------------------------- |
| `read<T>()`                                              | 将当前 Weight 数据读取成指定类型 `T`           |
| `read_as_float()` / `read_as_int32()` / `read_as_byte()` | 将内嵌的权重数据转成浮点 / 整数 / 字节表示     |
| `create(...)`                                            | 用于在 IR 中构建新的 WeightOp                  |
| `clone_bf16()` / `clone_f16()` / `clone_int()` 等        | 将现有 Weight 复制一份，并转换为其他存储格式   |
| `split(...)`                                             | 对当前权重按某个轴进行切分，用于模型并行、分片 |
| `update(...)`                                            | 将已有权重的数据更新为新的内容                 |

**InputOp**模型的输入节点，是整个 MLIR 图的入口之一，负责连接模型的输入信息与后续操作

- 基本声明，这里就是提供了一个额外的 `ShapeInterface` 接口，可以通过这个接口获取形状信息，然后在shape推导Pass中就可以自动调用

```
def Top_InputOp: Top_BaseOp<"Input", [
  DeclareOpInterfaceMethods<ShapeInterface>
  ]>
```

| 属性名                 | 类型                            | 说明                                              |
| ---------------------- | ------------------------------- | ------------------------------------------------- |
| `shape_tensor`         | `I64ArrayAttr` (optional)       | 表示这个输入是否影响模型 shape（类似动态 shape）  |
| `do_preprocess`        | `BoolAttr` with default `false` | 是否对输入做预处理（比如图像 resize、mean/scale） |
| `pixel_format`         | 自定义 `PixelFormatAttr`        | 像素格式，如 RGB、BGR                             |
| `channel_format`       | 自定义 `ChannelFormatAttr`      | 通道排列格式，如 NCHW、NHWC                       |
| `resize_dims`          | `I64ArrayAttr`                  | 目标 resize 大小，比如 `[224, 224]`               |
| `keep_aspect_ratio`    | `BoolAttr`                      | resize 是否保持原始宽高比                         |
| `keep_ratio_mode`      | `StrAttr`                       | 保比模式，比如填充、裁剪等                        |
| `pad_value`            | `I64Attr`                       | 填充时用什么值补齐（比如 0）                      |
| `pad_type`             | 自定义 `PadModeAttr`            | 填充方式，如 `CONSTANT`, `REFLECT` 等             |
| `scale`, `mean`        | `F64ArrayAttr`                  | 常规图像预处理：输入归一化用到的 mean 和 scale    |
| `customization_format` | `StrAttr`                       | 用户自定义预处理格式标记（特殊用途）              |
| `aligned`              | `BoolAttr`                      | 是否需要做内存对齐处理                            |



**TupleOp** 



```
let arguments = (ins
  Variadic<AnyTensor>:$inputs
);
```

- 输出 任意张量

**UnTupleOp**

- 输入项 接收任意多个张量作为输入  一般 `UnTupleOp` 的输入是一个 `TupleOp` 的输出

```tb
let arguments = (ins
  Variadic<AnyTensor>:$inputs
);
```

- 输出结果 任意张量

**功能性算子模板 Top_Op**

```
class Top_Op<string mnemonic, list<Trait> traits = []> :
    Top_BaseOp<mnemonic, !listconcat(traits,
       [DeclareOpInterfaceMethods<InferenceInterface>,
        DeclareOpInterfaceMethods<FlopsInterface>,
        DeclareOpInterfaceMethods<ShapeInterface>])>;
```

| 接口类名             | 实现功能                                 | 场景                 |
| -------------------- | ---------------------------------------- | -------------------- |
| `InferenceInterface` | 自动推理 Op 的输出类型                   | type inference       |
| `FlopsInterface`     | 统计该算子的 FLOPs（浮点计算量）         | 性能分析 / 优化      |
| `ShapeInterface`     | 提供 `getShape()` 等方法，返回张量 shape | 用于 shape inference |



**CastOp**

- arguments

```
let arguments = (ins
  AnyTensor:$input,
  DefaultValuedAttr<RoundModeAttr, "\"HalfAwayFromZero\"">:$round_mode
);
```



```
def RoundModeAttr : StrEnumAttr<"RoundMode", "Rounding strategy for quantization", [
  "HalfAwayFromZero",
  "HalfUp",
  "HalfDown",
  "HalfToEven"
]> {
  let cppNamespace = "::tpu_mlir::top";
}
```

每种属性则是代表了一种模式，比如HalfAwayFromZero0.5 → 1，HalfToEven0.5 → 最近的偶数，HalfUp 向上取整，HalfDown向下取整这些，这些则在float到int的过程中决定小数怎么变成整数

**CocatOp**

输入时任意长度的张量列表

```
let arguments = (
  ins Variadic<AnyTensor>:$inputs,
  DefaultValuedAttr<SI32Attr, "1">:$axis,
  DefaultValuedAttr<BoolAttr, "false">:$do_relu,
  DefaultValuedAttr<F64Attr, "-1.0">:$relu_limit,
  DefaultValuedAttr<RoundModeAttr, "\"HalfAwayFromZero\"">:$round_mode
);
```

属性

| 字段名       | 类型            | 默认值               | 说明                                        |
| ------------ | --------------- | -------------------- | ------------------------------------------- |
| `axis`       | `SI32Attr`      | `1`                  | 在哪个维度拼接                              |
| `do_relu`    | `BoolAttr`      | `false`              | 是否启用 ReLU 激活                          |
| `relu_limit` | `F64Attr`       | `-1.0`               | 如果为负数表示无 upper bound（仅普通 ReLU） |
| `round_mode` | `RoundModeAttr` | `"HalfAwayFromZero"` | 取整模式（用于数据量化时）                  |

- Canonicalize

为这个 Op 编写简化规则，比如连续两个 `ConcatOp` 沿同一个 axis 可合并成一个

**ConvOp**

- arguments

```
  let arguments = (ins
    AnyTensor:$input,
    AnyTensor:$filter,
    AnyTensorOrNone:$bias,
    I64ArrayAttr:$kernel_shape,
    I64ArrayAttr:$strides,
    I64ArrayAttr:$pads, // top,left,bottom,right
    DefaultValuedAttr<I64Attr, "1">:$group,
    OptionalAttr<I64ArrayAttr>:$dilations,
    OptionalAttr<I64ArrayAttr>:$inserts,
    DefaultValuedAttr<BoolAttr, "false">:$do_relu,
    DefaultValuedAttr<F64Attr, "-1.0">:$relu_limit,
    DefaultValuedAttr<BoolAttr, "false">:$dynweight_reorderd,
    DefaultValuedAttr<I64Attr, "1">:$weight_is_coeff,
    OptionalAttr<BoolAttr>:$do_winograd,
    OptionalAttr<AutoPadModeAttr>:$auto_pad, // only used in shape infer
    OptionalAttr<F64Attr>:$in_int4_scale,
    OptionalAttr<F64Attr>:$in_int4_zp,
    OptionalAttr<F64Attr>:$out_int8_scale,
    OptionalAttr<F64Attr>:$out_int8_zp
  );

```

卷积参数部分

| 参数           | 类型           | 含义                                              |
| -------------- | -------------- | ------------------------------------------------- |
| `kernel_shape` | `I64ArrayAttr` | 卷积核尺寸，如 [3, 3]                             |
| `strides`      | `I64ArrayAttr` | 步长，如 [1, 1]                                   |
| `pads`         | `I64ArrayAttr` | padding 信息，顺序为 `[top, left, bottom, right]` |
| `group`        | `I64Attr`      | 分组卷积参数，1 表示普通卷积                      |
| `dilations`    | `I64ArrayAttr` | 空洞卷积使用的 dilation 系数                      |
| `inserts`      | `I64ArrayAttr` | padding 内部扩展填充，较少使用                    |

算法控制参数

| 参数                 | 类型       | 含义                                     |
| -------------------- | ---------- | ---------------------------------------- |
| `do_relu`            | `BoolAttr` | 是否在卷积后接 ReLU                      |
| `relu_limit`         | `F64Attr`  | ReLU 最大阈值（-1 表示无）               |
| `dynweight_reorderd` | `BoolAttr` | 是否动态重排权重                         |
| `weight_is_coeff`    | `I64Attr`  | 是否视为系数权重（比如 depthwise）       |
| `do_winograd`        | `BoolAttr` | 是否启用 Winograd 卷积算法（适用于 3x3） |
| `auto_pad`           | 枚举       | 自动补零（如 `SAME`、`VALID`）           |

量化支持

| 参数             | 类型      | 用途              |
| ---------------- | --------- | ----------------- |
| `in_int4_scale`  | `F64Attr` | INT4 输入缩放因子 |
| `in_int4_zp`     | `F64Attr` | INT4 输入零点     |
| `out_int8_scale` | `F64Attr` | INT8 输出缩放因子 |
| `out_int8_zp`    | `F64Attr` | INT8 输出零点     |

- 输出类型也是任意张量
-  hasCanonicalizer Op优化规则
- extraClassDeclaration

```tb
let extraClassDeclaration = [{
  conv_attr_t parseParam();
  conv_attr_t dynparseParam();
}];
```

在自动生成的 C++ `top::ConvOp` 类中加入两个成员函数:

`parseParam()`：从 `Op` 提取出所有卷积参数，构造成 `conv_attr_t`（内部定义的结构体）；

`dynparseParam()`：可能用于运行时动态 shape 情况下的推理。



**AddOp**

- 定义头部

```
def Top_AddOp: Top_Op<"Add", [
  SupportFuseRelu,
  SupportConstant,
  ScalarProducer,
  ScalarConsumer
]> {

```

| Trait 名称        | 说明                     |
| ----------------- | ------------------------ |
| `SupportFuseRelu` | 表示此 Op 支持 ReLU 融合 |
| `SupportConstant` | 表示可以将输入常量化     |
| `ScalarProducer`  | 此 Op 可能生成标量结果   |
| `ScalarConsumer`  | 此 Op 也可以接受标量输入 |



- arguments

```
let arguments = (ins
  Variadic<AnyTensor>:$inputs,
  DefaultValuedAttr<BoolAttr, "false">:$do_relu,
  DefaultValuedAttr<F64Attr, "-1.0">:$relu_limit,
  OptionalAttr<F64ArrayAttr>:$coeff,
  DefaultValuedAttr<BoolAttr, "false">:$is_scalar
);

```

输入依旧是任意长度的张量类型

| 字段名       | 类型                 | 含义                                                         |
| ------------ | -------------------- | ------------------------------------------------------------ |
| `do_relu`    | BoolAttr，默认 false | 是否启用激活函数 ReLU                                        |
| `relu_limit` | F64Attr，默认 -1.0   | ReLU 的最大值，-1.0 表示不限制                               |
| `coeff`      | fF64ArrayAttr        | 每个输入张量的缩放系数，类似于 `output = sum(coeff[i] * input[i])` |
| `is_scalar`  | BoolAttr，默认 false | 如果是 scalar 模式，就把其中某个输入视为一个 float 值进行加法 |

- hasCanonicalizer 优化

输入为常量：可以直接进行计算并替换为 `top.Weight`

`Add` + `Relu` → `Add[do_relu=true]`（融合 ReLU）

只有一个输入且 coeff=1：可简化为 `Identity`



**SubOp**

- 定义头部

```
def Top_SubOp: Top_Op<"Sub", [
  SupportFuseRelu,
  SupportConstant,
  ScalarProducer,
  ScalarConsumer
]> {

```

| Trait 名称        | 说明                     |
| ----------------- | ------------------------ |
| `SupportFuseRelu` | 表示此 Op 支持 ReLU 融合 |
| `SupportConstant` | 表示可以将输入常量化     |
| `ScalarProducer`  | 此 Op 可能生成标量结果   |
| `ScalarConsumer`  | 此 Op 也可以接受标量输入 |

- arguments

```
let arguments = (ins
  Variadic<AnyTensor>:$inputs,
  DefaultValuedAttr<BoolAttr, "false">:$is_reverse,
  DefaultValuedAttr<BoolAttr, "false">:$do_relu,
  DefaultValuedAttr<F64Attr, "-1.0">:$relu_limit,
  OptionalAttr<F64ArrayAttr>:$coeff,
  DefaultValuedAttr<BoolAttr, "false">:$is_scalar
);

```

| 字段         | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| `inputs`     | 接收变长的任意张量输入                                       |
| `is_reverse` | 若为 true，则执行 `input2 - input1` 而非默认的 `input1 - input2` |
| `do_relu`    | 是否激活 ReLU                                                |
| `relu_limit` | 激活上限                                                     |
| `coeff`      | 输入系数缩放（如 `[1.0, 0.5]` 表示第一个输入不变，第二个输入乘 0.5） |
| `is_scalar`  | 输入是否是标量（广播场景）                                   |
