

- 验证Trait
- 接口Trait
- Pass 的 Trait

提供验证逻辑

Trait **最常见的作用**就是在 **`verify()` 阶段做验证**：

- 编译器读 `.mlir` 文件 → 遇到 `"my.add"`
- 查找对应的 `AddOp` 定义
- 调用 `AddOp::verify()`
	- 这里面会自动调用所有挂在这个 Op 上的 Trait 的 `verifyTrait()`
	- 比如 `SameOperandsAndResultType` 就会检查输入输出类型一致

如果不符合，编译器就报错，不会创建这个 Operation。

- 提供接口

比如shapeinfer()这些接口函数，实际上也是Trait生成的

如下InferShapeInterface实际上就是我们定义的接口**

```tablegen
def MyOp : Op<MyDialect, "myop",
              [ZeroOperands, OneResult, InferShapeInterface]> {
  ...
}
```

接口定义实际上是,对应的Op类能创建相应的接口方法：

```c++
def InferShapeInterface : OpInterface<"ShapeInference"> {
  let methods = [
    InterfaceMethod<"void", "shape_inference", (ins "InferenceParameter&":$param)>
  ];
}
```

比如上面说的

```
def MyOp : Op<MyDialect, "myop",
              [ZeroOperands, OneResult, InferShapeInterface]> {
  ...
}
```

我们当然会生成一个`MyOp`类，比如上面定义的InferShapeInterface类，自然就有了这个类里的接口方法，所以这就是Trait的本质

### 我们自己定义的Trait到底是什么？

**在TPU-MLIR项目里自己定义的Trait.h部分就完全是标记的意思**

**举例：**

- 看到下面这个Op定义，发现SupportFuseRelu这几个Trait标签既不是系统自带的验证trait，也不是我们自己写的接口trait，实际上他是我们自己写的标记Trait，下面讲

```
def Top_AddOp : Top_Op<"Add", [SupportFuseRelu, SupportConstant, ScalarProducer, ScalarConsumer]> {
  ...
}
```

- 我们查阅Traits.h发现里面其中就定义了这些trait标签，，可以看到这里啥也没写，仅仅继承了 `NativeOpTrait<name>`，这个功能也就是其实没什么功能，说明当前定义的是一个Trait而已，然后就可以挂在Op上了，这样就会对应生成c++基类，然后来供Op继承使用

```
class TPUMLIROpTrait<string name> : NativeOpTrait<name> {
  let cppNamespace = "::tpu_mlir::trait";
}

def ScalarProducer : TPUMLIROpTrait<"ScalarProducer">;
def ScalarConsumer : TPUMLIROpTrait<"ScalarConsumer">;
def SupportConstant: TPUMLIROpTrait<"SupportConstant">;
def SupportFuseRelu : TPUMLIROpTrait<"SupportFuseRelu">;
```

- 这里仅仅提供的是从cppNamespace得到的，就是这些类会在这个命名空间内，比如生成的：

```c++
namespace tpu_mlir::trait {
  class ScalarProducer : public NativeOpTraitBase { };
  class ScalarConsumer : public NativeOpTraitBase { };
}
```



- 但是很明显这里定义的时候并没有写任何其他的内容，所以生成的内容就是基本上啥也没有的，仅仅就是一个基类，然后给挂载的Op使用



- 继续往下走，就是这个标记的功能到底是什么意思，有时怎么获取到的标记呢

前面我们已经说过，这些Trait挂载在Op上，然后生成的c++代码种Op会继承这写trait类，生成的代码如下（就是间接继承的内容）

```c++
class Top_AddOp : public Top_Op<Top_AddOp, SupportFuseRelu, ScalarProducer> { };
```

- 然后既然说的是标签标记，怎么识别到呢？ 用下面这个方法   
- 这个 `hasTrait<T>()` 底层实现其实就是检查 Op 的模板继承类型里有没有 `T`，或者使用 `dynamic_cast<T*>(op)` 判断对象是否能向上转换成 Trait 类型，因为Op都继承了这些Trait类，所以肯定是向上继承的，能向上转换就起到了标记的作用

```c++
if (op->hasTrait<tpu_mlir::trait::ScalarProducer>()) {
    // op 是一个 ScalarProducer
}
```



**实际上这是c++中一个关于类的使用方式， 叫做标记类，可以复习一下**