# 调用过程

**注册的效果：只有通过注册之后才能通过命令行调用具体的Pass命令**

- 调用了regiterAllPasses()这个函数，自己写的，相当于对注册的所有相关Pass进行了一层封装

```
void registerAllPasses() {
  registerCanonicalizer();
  mlir::registerConversionPasses();
  top::registerTopPasses();
  tpu::registerTpuPasses();
}
```

这里封装了四个Pass的注册

`registerCanonicalizer()`

```
tpuc-opt xxx.mlir --canonicalize
```

 `mlir::registerConversionPasses()`

```
mlir-opt test.mlir --convert-linalg-to-affine
```

`registerTopPasses()`

 `tpu::registerTpuPasses()`

- 进入top::registerTopPasses()，发现其结构如下

**里面是对其他具体的Pass的注册**

**因为Top相关的Pass可能有很多个**

```c++
inline void registerTopPasses() {
  registerAddPostprocess();
  registerDeinit();
  registerExtraOptimize();
  registerFusePreprocess();
  registerImportCalibrationTable();
  registerInit();
  registerProcessorAssign();
  registerProcessorOptimize();
  registerPruning();
  registerQDQConvert();
  registerShapeInfer();
}

```



```c++
inline void registerAddPostprocess() {
  ::mlir::registerPass([]() -> std::unique_ptr<::mlir::Pass> {
    return createAddPostprocessPass();
  });
}

```

`::mlir::registerPass(...)`

这是 MLIR 提供的 API，用来动态注册一个匿名 Pass

`[]() -> std::unique_ptr<::mlir::Pass>`

这是一个 lambda 函数

`createAddPostprocessPass()`返回一个 `AddPostprocessPass` 类的实例，封装了真正要执行的逻辑

- 我们继续往下走，看一下工厂函数内部（因为这部分是我们自己写的，但是registerPass()那部分并不是自己写的，所以跳过registerPass() 那边，然后直接看工厂函数那边，我们暂时会用registerPass()就可以）

```c++
std::unique_ptr<OperationPass<ModuleOp>> createAddPostprocessPass() {
  return std::make_unique<AddPostprocessPass>();
}
```

- 类里的内容是自己写的重点，相当于前面部分是td文件生成或者有固定的模板，这部分才是我们自己要写的内容

**这里很明显就是标准的Pass组成，构造函数，然后一个必须实现的runOnOperation()方法，然后是一些声明的其他方法（这部分直接在类外面实现了），以及成员变量**

**在runOnOperation()的逻辑里则是不断地调用自身Pass类的其他方法，来实现具体的Pass功能**

```c++
class AddPostprocessPass : public AddPostprocessBase<AddPostprocessPass> {
public:
  AddPostprocessPass() {}
  void runOnOperation() override {
    post_type = this->type;
    if (post_type.empty()) {
      return;
    }
    auto mOp = getOperation();
    auto func = module::getMainFuncOp(mOp);
    terminator = func.getBody().back().getTerminator();
    auto arg_type = func.getArgumentTypes();
    in_shape = arg_type[0].cast<RankedTensorType>().getShape();
    batch = in_shape[0];
    height = in_shape[2];
    width = in_shape[3];
    topk = 200;
    // insert posthandleOp
    OpBuilder builder(&getContext());
    builder.setInsertionPoint(terminator);
    if (post_type.ends_with("seg")) {
      insertYolosegOp(builder);
    } else if (post_type.starts_with("yolo")) {
      insertYoloOp(builder);
    } else if (post_type == "ssd") {
      insertSsdOp(builder);
    } else if (post_type == "bnr") {
      insertDepackRawOp(builder);
    } else if (post_type == "mmap2rgbmap") {
      insertMmap2RgbmapOp(builder);
    }
    module::updateModuleTypes();
    module::setPostprocess(post_type);
  }

protected:
  void getYoloOperandsAndAnchors(std::vector<Value> &operands,
                                 std::vector<double> &anchors);
  void insertYoloOp(OpBuilder &builder);
  void insertYolosegOp(OpBuilder &builder);
  void insertSsdOp(OpBuilder &builder);
  void insertDepackRawOp(OpBuilder &builder);
  void insertMmap2RgbmapOp(OpBuilder &builder);

protected:
  std::vector<int64_t> in_shape;
  int64_t batch, height, width, topk;
  StringRef post_type;
  Operation *terminator;
  StringRef intermode, intercoordmode;
};
```
