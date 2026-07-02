**Attribute**

### 1、是什么

```mlir
%0 = "arith.constant"() {value = 42 : i32} : () -> i32
```

这里的 `42 : i32` 就是一个 Attribute（整型常量）

键值对这种也属于一个大的attribute，内部由键和值组成，适用于那种表示Op的attribute（就是写在Op后面的，一般都是这种，下面是两种Op举例）

**常见**

```
%0 = "arith.constant"() {value = 42 : i32} : () -> i32
```

**直接作为常量,`addi` 属于 arith dialect**

```
%0 = addi 40, 2
```



### 2、attribute

键值对大括号，这个本身就是mlir的语法，然后大括号内部的内容则是具体的attribute，比如0.1 : f32是一个具体的 Attribute

```mlir
{alpha = 0.1 : f32, beta = 0.2 : f32}
```

整数常量

`{value = 42 : i32}` 整体是一个 Attribute

其中的 `value` 是字典里的 **键名**

```
{value = 42 : i32}
```

- 浮点常量

```
{value = 3.14 : f64}
```

- 字符串常量

```
{label = "hello"}
```

- 数组

```
{dims = [1, 2, 3]}
```


- 把 Type 包装成一个 Attribute

```
{dtype = f32}
```

- 只有存在与否的属性

```
{flag}
```

### 3、划分

```
%0 = addi 40, 2
```

它看到 `40` 和 `2` 就能直接算 `42`。

每个 Attribute 都有对应的 C++ 类：

| MLIR 文本表示                      |
| ---------------------------------- |
| `42 : i32`                         |
| `3.14 : f64`                       |
| `"hello"`                          |
| `{value = 42 : i32}`               |
| `dense<[1, 2, 3]> : tensor<3xi32>` |



**在 `.mlir` 文件里Attribute 只是文本；在 MLIR 内部每种 Attribute 都有对应的强类型 C++ 类 Pass 能直接操作数值，而不用自己解析字符串。**

- `42 : i32` 
- `3.14 : f64` 
- `"hello"` 
- `{...}` 

这样在编译器 Pass 里，你就可以写：直接得到 C++ 的 `int`，而不是自己去解析字符串 `"42"`。

```cpp
if (auto intAttr = attr.dyn_cast<IntegerAttr>()) {
    int val = intAttr.getInt();
}

```



### 4、自定义Attribute

MLIR 已经内置了很多 Attribute，比如：

- `42 : i32`
- `3.14 : f64`
- `"relu"`

在实际项目（比如 TPU-MLIR）里常常需要表达一些**更高层语义信息**，内置的 Attribute 不够。

比如激活函数的类型选择是不是relu还是其他的选择

```
%0 = "top.add"(%a, %b) {activation = "relu"} : ...
```



### 5、自定义Attribute

比如我们要定义一个 **枚举属性**：

```

include "mlir/IR/AttrTypeBase.td"

def Top_Dialect : Dialect {
  let name = "top";
  let cppNamespace = "::tpu_mlir::top";
}

def ActivationAttr : EnumAttr<Top_Dialect, "activation",
    ["relu", "sigmoid", "tanh"]> {
  let summary = "Activation function attribute";
  let cppNamespace = "::tpu_mlir::top";
}
```

编译

```cpp
class ActivationAttr : public mlir::Attribute {
public:
  static ActivationAttr get(mlir::MLIRContext *ctx, Activation value);

  Activation getValue() const;
};
```

- 在Op

```tablegen
def Top_AddOp : Op<Top_Dialect, "add"> {
  let arguments = (ins AnyTensor:$lhs, AnyTensor:$rhs);
  let results   = (outs AnyTensor:$output);

  let attributes = (ins
    ActivationAttr:$activation // 新增属性
  );
}
```



