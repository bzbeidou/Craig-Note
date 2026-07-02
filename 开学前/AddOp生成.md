#### 1、Op   详解定义的通过tablegen语法生成的Op的c++定义（以AddOp为例）

```c++
namespace mlir {
namespace north_star {
class AddOp;
}
}
```

```c++
#ifdef GET_OP_CLASSES
#undef GET_OP_CLASSES
```

**名为`detail`的命名空间**

```c++
namespace mlir {
namespace north_star {
namespace detail {
    class AddOpGenericAdaptorBase { ... }
}
template <typename RangeT>
class AddOpGenericAdaptor : public detail::AddOpGenericAdaptorBase { ... }   
class AddOpAdaptor : public AddOpGenericAdaptor<::mlir::ValueRange> {...}
class AddOp : public ::mlir::Op<AddOp, ::mlir::OpTrait::ZeroRegions, ::mlir::OpTrait::OneResult, ::mlir::OpTrait::OneTypedResult<::mlir::Type>::Impl, ::mlir::OpTrait::ZeroSuccessors, ::mlir::OpTrait::NOperands<2>::Impl, ::mlir::OpTrait::OpInvariants> {...}
} // namespace north_star
} // namespace mlir
MLIR_DECLARE_EXPLICIT_TYPE_ID(::mlir::north_star::AddOp)
```

** AddOpGenericAdaptorBase  `Adaptor`**

提供基本的 getter

- `getAttributes()`
- `getProperties()` 

**一句话：泛型版本的Adaptor，用来处理任意容器 **

通过 `getLhs()`, `getRhs()` 访问第一个和第二个参数

`getOperands()` 获取整体输入

具体版本的 `Adaptor`，用于 `ValueRange` 类型

**MLIR_DECLARE_EXPLICIT_TYPE_ID(AddOp)`**

MLIR中用于在类型系统中注册 `AddOp` 的宏，方便做类型推断和判断。

**总结：**

 **通过 TableGen 自动生成的** 对 `AddOp` 的完整 C++ 封装，它包含：

- 操作本身的类（AddOp）
- 参数访问器（Adaptor）
- 构造和校验方法（build/verify）
- 泛型支持（GenericAdaptor）

它使得可以在 C++ 中像使用类一样使用 MLIR 中的 Op

#### 2、 mlir OP的一些知识点

**2.1 Adaptor**是 Op 的轻量访问器类，用于在没有构造完整 Op 对象的情况下，访问其操作数、属性、Region 等数据

在 `AddOp` 的 TableGen 中，如果定义了操作数、属性，MLIR 会自动为你生成 `AddOpAdaptor`, 为了方便访问 operand/attr 时不构造整个 Op**目前先知道Adaptor是只读不写的一个访问类即可，也就是能读到自己定义的Op的内容，但是不能改变它**

定义了一个 Op，比如：

```
def AddOp : NS_Op<"add", [Pure]> {
  let arguments = (ins I32:$lhs, I32:$rhs);
  let results = (outs I32);
}

```

这在 C++ 中会生成一个类 `AddOp`，它很强大，可以访问一切内容。但这个类必须绑定到一个 IR 中的 Operation 实例

```cpp
AddOp op = ... // 你必须已经有一个 Op
Value lhs = op.getLhs();  // OK
```

但有些场景你只是拿到了一个 `mlir::Operation*`，比如：

```c++
mlir::Operation* rawOp = ...;
```

TableGen 为自动生成了：

```
class AddOpAdaptor {
public:
  AddOpAdaptor(mlir::ValueRange operands, mlir::DictionaryAttr attributes);
  
  Value getLhs();
  Value getRhs();
  IntegerAttr getSomeAttr();
  LogicalResult verify(...);  // 可选：校验属性合法性
};

```



#### 3、自己写的op

**3.1 注册Op**

```c++
void NorthStarDialect::registerOps() {
  llvm::outs() << "register " << getDialectNamespace() << "  Op\n";
  addOperations<
#define GET_OP_LIST
#include "Dialect/NorthStar/IR/NorthStarOps.cpp.inc"
      >();
}

```

通过 TableGen 定义的所有 Op 注册到 NorthStar Dialect 中，让它们可以被 MLIR 识别和使用。

```c++
addOperations<
#define GET_OP_LIST
#include "Dialect/NorthStar/IR/NorthStarOps.cpp.inc"
>();

```

首先里面有一个条件宏#define GET_OP_LIST，也就是一个开关之前讲过，配合这边很容易理解条件宏的概念，直接看.inc文件这里的部分

展开如下,也就是把所有的Op都给包含进来了

```
// 展开后大概会变成这样：
addOperations<
  mlir::north_star::AddOp,
  mlir::north_star::MulOp,
  mlir::north_star::PrintOp
>();

```

从tablegen定义，然后会自动生成这些条件宏的代码

addOperations<>();就可以了

`addOperations<...>()` 是一个模板函数（泛型函数），是 **`mlir::Dialect` 的成员函数**，你自定义的 `NorthStarDialect` 是从它继承的。

源码可以了解一下,可以看到里面实际上使用了addOperation这个函数，没加s

```c++
template <typename... OpTs>
void addOperations() {
  // 展开参数包，逐个注册
  (void)std::initializer_list<int>{
      0, (addOperation(OpTs::getOperationName(), OpTs::getOperationInfo()), 0)...};
}
```



