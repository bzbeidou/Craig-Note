## 1、Pass用来做什么？

Pass就是用来改造IR的，比如有一个.mlir文件，其中的内容可能是“top.add”，通过Pass将其可以转换成tpu层之下的

**也就是对IR的改造通常都是Pass**，可以分为分析类Pass以及变换类，这里的分析也可能是Pass

**分析类** Pass：不改动 IR，只读取信息

**变换类** Pass：会改动 IR



## 2、是什么？  

- `mlir::PassWrapper<MyPass, OperationPass<FuncOp>>`
	- `MyPass`：你定义的类名
	- `FuncOp`：这个 Pass 作用的 IR 层级（Module / Func / Region）

```c++
struct MyPass : public mlir::PassWrapper<MyPass, OperationPass<FuncOp>> {
  void runOnOperation() override {
    auto func = getOperation();
  }
};
```

1. 实现 `runOnOperation()` 或 `runOnModule()`
2. 注册 Pass 名称、描述信息
3. 可选择接受参数

## 3、类的内部

**（1）直接操作：**

Pass则可以调用Op的接口方法，来进行一定的操作

getOps可以获取到其中的Op，然后可以修改属性等等操作，

```
for (auto &op : func.getOps<top::AddOp>()) {
}

```

**（2）使用pattern**

pattern就是匹配+重写的组合，可以批量改造IR，既可以对Op进行折叠优化，也可以实现从top到tpu dailect切换

Pass 可以注册 Pattern，然后调用 `applyPatternsAndFoldGreedily` 去自动匹配 IR 并应用规则。


