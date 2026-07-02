

比如这是Trait的文件内容，在Trait.td文件里，然后我们在同文件目录下也写了如下的Traits.h文件，如下

```
class TPUMLIROpTrait<string name> : NativeOpTrait<name> {
  let cppNamespace = "::tpu_mlir::trait";
}

def ShapeProducer : TPUMLIROpTrait<"ShapeProducer">;
```

下面是.h文件的内容，我们已经知道了这种定义的标记性的Trait就是对应的空类，但是不会自动生成，而是需要我们写如下内容 **很明显，自己定义了这个空类**

```c++
template <typename ConcreteType>
class ShapeProducer
    : public ::mlir::OpTrait::TraitBase<ConcreteType, ShapeProducer> {};
```



这是标记类trait

```
def ShapeProducer : TPUMLIROpTrait<"ShapeProducer">;
```

这是Op的定义，可以看到这里就用到了前面定义的ShapeProducer内容，可以说这里仅仅就是说明我定义了这个空类，AddOp可以继承，但是具体的空类还是得我们自己写的，否则找不到

```
def AddOp : Op<"AddOp", [ZeroOperands,ShapeProducer]>{}
```

TableGen会在生成的Op类里展开：

```c++
class AddOp : public mlir::Op<AddOp,
                              mlir::OpTrait::ZeroOperands,
                              tpu_mlir::trait::ShapeProducer> {
   ...
};
```

**内置 Trait（MLIR 自带）**

- 比如：
	- `mlir::OpTrait::ZeroOperands`
	- `mlir::OpTrait::OneResult`
	- `mlir::OpTrait::SameOperandsAndResultType`
	
	
	
- 当你在 TableGen 里写 `[ZeroOperands, OneResult]`，TableGen 展开时，C++ 代码里直接继承这些现成的类。

**用户自定义 Trait**

- 比如：
	- `SupportFuseRelu`
	- `SupportConstant`
	- `ShapeProducer`
	- `ScalarConsumer`