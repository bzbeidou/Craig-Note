比如我在 `$inputs` / `$is_scalar` 有时会自动提示

**不是预定义的关键字**，而是在 **TableGen 里起的名字**。

$xxx` 部分，就是给这个 operand/attr 起名字。

- TableGen 会用这个名字生成对应的方法**。

**举例：**

```
ins Variadic<AnyTensor>:$inputs
```

生成的C++代码大致如下

```c++
::mlir::OperandRange getInputs();
```

再比如

```
DefaultValuedAttr<BoolAttr, "false">:$is_scalar
```

生成的c++代码如下

```c++
::mlir::BoolAttr getIsScalarAttr();
bool getIsScalar();   // 自动应用默认值
```
