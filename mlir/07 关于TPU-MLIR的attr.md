#### 1、需求

很多时候，属性的取值只能在 **有限的字符串集合** 里，比如：Pooling 的 mode 只能是 `"Max"` 或 `"Avg"`;ArgMode 只能是 `"ArgMin"` 或 `"ArgMax"`。

如果仅仅使用`StringAttr`，那么用户可能随便写`Hello`, 也能被读取.mlir的时候直接拿出来，但是实际上这是不应该的。

所以有了这个需求就是去设计限定版的`StringAttr`，如下

#### 2、TPU-MLIR的例子

```
class AnyStrAttrOf<list<string> cases>
    : StringBasedAttr<
          CPred<!foldl(
              "$_self.cast<StringAttr>().getValue() == \"" #!head(cases) #"\"",
              !foreach(
                  case, !tail(cases),
                  "$_self.cast<StringAttr>().getValue() == \"" #case #"\""),
              prev, cur, prev #" || " #cur)>,
          "string attribute whose value is " #!foldl(
              /*init*/ !head(cases), /*list*/ !tail(cases), prev, cur,
              prev #", or " #cur)>;

// 定义一个具体的属性：ArgModeAttr
def ArgModeAttr: AnyStrAttrOf<["ArgMin","ArgMax"]>;

```

- 一是`AnyStrAttrOf`

 `StringBasedAttr`，基于字符串的属性。

- 一个`def ArgModeAttr: AnyStrAttrOf<["ArgMin","ArgMax"]>`

`StringBasedAttr<Predicate, Summary>` 里，第一个参数就是 **验证条件**，TableGen 提供了多种 Pred 封装：`CPred<"...">`就是其中的一种，然后可以写C++的验证逻辑，然后会把这段逻辑嵌入到生成的c++代码里

- CPred写的部分

比如我写了如下代码

```
CPred<"$_self.cast<StringAttr>().getValue() == \"ArgMin\" ||
       $_self.cast<StringAttr>().getValue() == \"ArgMax\"">
```

展开之后再c++的.inc文件里会出现如下代码

```c++
/// Verify that ArgModeAttr is valid.
::mlir::LogicalResult ArgModeAttr::verify(
    ::mlir::Attribute attr, ::mlir::function_ref<void(::mlir::InFlightDiagnostic &&)> emitError) {
  if (!attr.isa<mlir::StringAttr>() ||
      !(attr.cast<StringAttr>().getValue() == "ArgMin" ||
        attr.cast<StringAttr>().getValue() == "ArgMax")) {
    return emitError(::mlir::emitError(attr.getLoc(), "invalid ArgModeAttr"));
  }
  return ::mlir::success();
}

```

#### 3、实际

- 定义

```tablegen
def MyArgOp : Top_Op<"myarg", []> {
  let arguments = (ins ArgModeAttr:$mode);
}
```

- 这个算子这样写：

```mlir
%0 = "top.myarg"() {mode = "ArgMin"} : () -> tensor<4xf32>
%1 = "top.myarg"() {mode = "ArgMax"} : () -> tensor<4xf32>
```

```
%2 = "top.myarg"() {mode = "SomethingElse"} : () -> tensor<4xf32>
```

#### 4、补充

C++ 条件片段，只是一段字符串，它的作用就是代表一个**谓词（Predicate）**，叫做验证逻辑就可以，写的内容会对应生成到相应的verify逻辑里

不止能写在Attribute定义里，还能写在很多需要用到谓词验证的地方，下面列举几种常见用途。

Attr定义里，就是我们上面引入的时候用的

Type定义可以在Type的td定义里放上CPred，用来限制某种类型的合法性。结果声称在Type的verify方法里

Op参数（Operands/Results/Attributes），在定义Op的参数的时候也可以用`CPred`,比如限制操作数必须是某种形状或者类型，结果就是生成在Op::Verify()里。