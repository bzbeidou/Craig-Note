#### 1、Operation

**（1）Operation定义**

可以看作c++中的函数，有函数名、输入参数、返回值等等

| 概念        | C++ 函数/函数调用            | MLIR 中的 Operation (Op)  |
| ----------- | ---------------------------- | ------------------------- |
| 名称        | `add()`                      | `toy.add`                 |
| 输入参数    | `add(a, b)` 中的 `a`, `b`    | `toy.add %a, %b`          |
| 返回值      | `int c = add(a, b)` 中的 `c` | `%c = toy.add ...`        |
| 函数体/定义 | 有函数体 `{ return a + b; }` | 没有（Op 是一个语义单位） |

**（2）怎么用**

下面有三个语句，实际上就是三个Op，分别使用了toy.constant\toy.add这两个定义的Op

```
%a = toy.constant 3 : i32
%b = toy.constant 4 : i32
%c = toy.add %a, %b : i32
```

*也就是理解为mlir中一句就是一个Op，定义的部分也是叫做Op，这两个没区别，一个是使用、一个是定义*

**（3）mlir是Op组成**

比如：

算术运算：`toy.add`, `toy.sub`, ...

常量定义：`toy.constant`

函数返回：`toy.return`

控制流语句：`scf.if`, `scf.for`, ...

调用函数：`func.call`

自定义行为：自己定义的 `mydialect.custom_op`



**（4）如何定义一个Op**

以toy.add为例介绍如何定义一个Op

**第一 在.td文件中声明Op**

```
def Toy_AddOp : Toy_Op<"add", [Pure]> {
  let summary = "Addition of two toy values";
  let arguments = (ins F64:$lhs, F64:$rhs);
  let results = (outs F64:$result);
  let assemblyFormat = "$lhs `,` $rhs `:` type($result)";
}
```

Toy_Op是定义的Op模板类，表示定义的这个Op继承自资格Toy_Op(这个是官方给的？)

“add”,Pure这些是参数，有对应生成c++的作用，也是Toy_Op的一些参数

"arguments"和“result”则是模板类里的一些字段。被继承然后赋值，这里用于声明输入输出类型

assemblyFormat则是说明在.mlir中这个Op应该怎么写

**第二 配置CMake，生成Op定义代码**

```
set(TOY_OP_TD
  include/Toy/ToyOps.td
)

mlir_tablegen(ToyOps.h.inc -gen-op-decls)
mlir_tablegen(ToyOps.cpp.inc -gen-op-defs)

add_public_tablegen_target(ToyOpsIncGen)

```

**第三 生成c++代码   会自动生成**

```
// 自动生成的
class AddOp : public Op<AddOp, OpTrait::NOperands<2>::Impl,OpTrait::OneResult> {
public:
  Value getLhs();  // 获取输入
  Value getRhs();
  static StringRef getOperationName() { return "toy.add"; }
};

```

**第四 在Dialect中注册Op  这部分是在Dialect里定义的**

```c++
//在ToyDilect.cpp里

#include "Toy/ToyOps.h"

ToyDialect::ToyDialect(MLIRContext *ctx) : Dialect("toy", ctx, TypeID::get<ToyDialect>()) {
  addOperations<
#define GET_OP_LIST
#include "Toy/ToyOps.cpp.inc"
  >();
}

```

**（5）两类模板**

  很多Op很像，只是名字不同、输入个数不同等，因此定义模板可以减少工作量，比如要定义 10 个类似 `add/sub/mul/div/max/min/...` 的操作，如果每个都手写：

```
def NorthStar_AddOp : NorthStar_Op<"add", [...traits...]> {
  let arguments = (ins F32:$lhs, F32:$rhs);
  let results   = (outs F32:$res);
}
```

那么很容易发现这里输入个数是一样的，那么可以定义一个模板,也就是用于二元输入的Op定义，那么一元输入的可以另外定义一个模板

```
class NorthStar_BinaryOp<...> : NorthStar_Op<...> {
  let arguments = !con((ins OperandType:$lhs, OperandType:$rhs), attributes);
  let results = (outs resultType:$result);
}
```

接下来就方便了，不管定义add还是sub都可以引用这个模板，只需要在具体的def里传参就可以

```
def NorthStar_AddOp : NorthStar_BinaryOp<"add", F32, F32, [Pure]> {
  let summary = "Floating point addition";
}
```



**（5）ConstOp无模板**

```
def NorthStar_ConstOp : NorthStar_Op<"const", []> {
  let arguments = (ins ElementsAttr:$value);
  let results = (outs AnyNSTensor:$result);
}
```

#### 2、Tablegen 语法

**（1）允许定义模板类并传入参数**

```
class NorthStar_Op<string mnemonic, list<Trait> traits = []>
```

**（2）字符串拼接#**

```
let summary = cppNamespace#opName#"op";


会把这些部分拼接成最终字符串，例如
NorthStarAddOp // 如果 namespace 是 NorthStar，opName 是 Add

```





