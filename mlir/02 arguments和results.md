### argument和result一些内容

`argument`就是操作的输入描述，分为两类，可以看下面这个例子

**operands**

- 例如 `$inputs`（张量输入）
- 对应 IR 里的 `%0, %1, ...` 这些值
- `Operation.operands`

**attributes**

- 例如 `$do_relu`, `$relu_limit`
- 常量参数
-  `Operation.attrs`

```td
  let arguments = (ins
    Variadic<AnyTensor>:$inputs,
    DefaultValuedAttr<BoolAttr, "false">:$do_relu,
    DefaultValuedAttr<F64Attr, "-1.0">:$relu_limit,
    OptionalAttr<F64ArrayAttr>:$coeff,
    DefaultValuedAttr<BoolAttr, "false">:$is_scalar
  );
```

**两个方面原因**

- 首先是将arguments和result便于区分
- 其中是带有一定信息的 

在MLIR里每个Operation大致长这样

```c++
class Operation {
  std::vector<Value> operands;   
  std::vector<Value> results;   
  DictionaryAttr attrs;          
};
```

很明显，vector里存的都是Value类型

而且Value不是简单的数据，每个Value内部会记录

**暂时理解为为了方便写以及分析才把arguments和result分开设计的就可以，这边不用懂太多**

**（1）Variadic**

```
ins Variadic<AnyTensor>:$inputs
```

生成一个方法，然后这个方法的特点是自动用范围来存输入，也就是读到了这个Op之后（这个arguments在这个Op之内），当调用这个方法的时候存储就是自动用范围来存的，如下就是生成的内容。所以这句tablegen代码的作用就是生成这个方法，可以读取多个输入。

```c++
// 在 AddOp 这个类里生成
::mlir::OperandRange getInputs();
```

**（2） DefaultValueAttr**

- 语法

```
DefaultValuedAttr<BoolAttr, "false">:$do_relu
```

这个DefaultValuedAttr的内容就是生成一个c++方法，然后可以获取到输入.mlir的属性，然后如果没有这个属性的内容就提供一个默认的返回内容，就是"false"，这就是这个DefaultValuedAttr的作用。然后可以和OptionalValueAttr的内容对比着看。

**（3）OptionalAttr**

**区别于 DefaultValuedAttr**：如果调用下面这个方法去获取它的coeff属性，直接返回null。即使读取到了.mlir的AddOp里没有，那么也会默认返回false，这就是区别

这里生成的c

```c++
::mlir::ArrayAttr getCoeffAttr();
std::optional<mlir::ArrayAttr> getCoeff();
```

