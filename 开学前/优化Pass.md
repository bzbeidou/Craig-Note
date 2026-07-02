## 1、Pass

Pass就是一个优化或者转换步骤，输入一个IR，输出一个修改后的IR

**每个Pass都是一个单独的步骤，不可能一个Pass包含多个功能**

### 1.1 mlir中怎么定义和使用Pass

MLIR 提供了 Pass 管理器 `mlir::PassManager`，你可以这样用：

```mlir
mlir::PassManager pm(&context);
pm.addPass(mlir::createInlinerPass());            // 函数内联
pm.addPass(mlir::createCanonicalizerPass());      // 标准化简化（常量折叠）

if (failed(pm.run(module))) {
  llvm::errs() << "优化失败\n";
}

```

这部分是Pass执行逻辑，不是写在main函数里的

**这部分结构的代码在哪边呢？ 是作为另外的c++文件存在的**

```
my-project/
├── tools/
│   └── mymlir-opt/
│       ├── mymlir-opt.cpp       ← 入口函数（main）
├── lib/
│   └── MyPass/
│       ├── MyOptimizationPass.cpp  ← 实现具体 Pass 逻辑
│       └── MyOptimizationPass.h    ← 声明 Pass 接口
```



162 1// =======================2// 1. 计算 FLOPs3// =======================4int64_t top::AddOp::getFLOPs() {5  // 总 FLOPs = 元素数 × (输入数量 - 1)6  // 例如两个输入：每个元素做一次加法7  // 如果有 ReLU，再加一次激活操作8  return module::getNumElements(getOutput()) *9         (getInputs().size() - 1 + (getDoRelu() ? 1 : 0));10}11​12// =======================13// 2. 初始化14// =======================15LogicalResult top::AddOp::init(InferenceParameter &p) {16  // 如果是两个输入，可以走 DNNL Binary 算子路径17  if (getInputs().size() == 2) {18    auto binary = new Binary();  // 创建一个 DNNL Binary 加法对象19    20    // 注意：这里原本有 Binary 对象的参数配置代码（被注释掉了）21    // 实际运行时会在 inference() 中再配置一次22​23    p.handle = (void *)binary; // 保存指针到 handle，供推理时使用24  } else {25    // 多输入加法走手写路径，不用 DNNL26    p.handle = nullptr;27  }28  return success();29}30​31// =======================32// 3. 资源释放33// =======================34void top::AddOp::deinit(InferenceParameter &p) {35  if (p.handle != nullptr) {36    auto binary = (Binary *)p.handle;37    delete binary;     // 释放 Binary 对象38    p.handle = nullptr;39  }40}41​42// =======================43// 4. 推理实现44// =======================45LogicalResult top::AddOp::inference(InferenceParameter &p) {46  // 推导广播后的输出 shape，并更新到 output47  auto output_shape = computer_broadcast_shape(getOperation());48  module::setShape(getOutput(), output_shape);49​50  if (getInputs().size() == 2) {51    // 双输入路径 —— 用 DNNL Binary 加速52    if (p.handle == nullptr) {53      return failure(); // 初始化时没创建 Binary，直接报错54    }55​56    auto binary = (Binary *)p.handle;57​58    auto lhs_shape = module::getShape(getInputs()[0]);59    auto rhs_shape = module::getShape(getInputs()[1]);60​61    // 配置 Binary 算子62    (*binary)63        .hs(p.inputs[0], p.inputs[1], lhs_shape, rhs_shape)  // 设置左右输入64        .dst(p.outputs[0], module::getShape(getOutput()))    // 设置输出缓冲区65        .do_relu(getDoRelu())                                // 是否做 ReLU66        .relu_limit(getReluLimit().convertToDouble())        // ReLU 上限67        .algorithem(algorithm::binary_add)                   // 算法类型 = 加法68        .setup();                                            // 完成配置69​70    binary->run(); // 执行加法运算71​72  } else {73    // 多输入路径 —— 手写并行加法74    auto num_elem = module::getNumElements(getOutput());75#pragma omp parallel for schedule(static, omp_schedule(num_elem))76    for (int64_t i = 0; i < num_elem; i++) {77      p.outputs[0][i] = 0;78      for (auto in : p.inputs) {79        if (in != nullptr) {80          p.outputs[0][i] += in[i];81        }82      }83    }84    // 如果开启了 ReLU85    if (getDoRelu()) {86      auto limit = getReluLimit().convertToDouble();87      function_relu(p.outputs[0], p.outputs[0], num_elem, limit);88    }89  }90​91  return success();92}93​94// =======================95// 5. 形状推导96// =======================97void top::AddOp::shape_inference() {98  // 1. 广播推导基础 shape99  broadcast_shape_inference(getOperation());100  bool has_scalar = false;101  for (int i = 0; i < getNumOperands(); i++) {102    auto value = getInputs()[i];103    broadcast_tensor_reshape(getOutput(), value); // 对标量或广播输入 reshape104    has_scalar = has_scalar && module::isScalar(value.getDefiningOp());105  }106​107  // 2. 判断是否需要 Shape Value Inference（即推导 shape 张量的数值）108  auto inputs = getInputs();109  bool need_shape_val_infer =110      std::all_of(inputs.begin(), inputs.end(),111                  [](auto in_op) {112                    return module::isWeight(in_op) || module::isShape(in_op);113                  }) &&114      std::any_of(inputs.begin(), inputs.end(),115                  [](auto in_op) { return module::isShape(in_op); });116  117  need_shape_val_infer &=118      std::any_of(inputs.begin(), inputs.end(),119                  [](auto in_op) { return module::isShape(in_op); });120​121  // 3. 标记输出是否为标量122  auto out_shape = module::getShape(getOutput());123  if (out_shape.size() == 1 && out_shape[0] == 1 && has_scalar) {124    auto context = getContext();125    mlir::Builder builder(context);126    setIsScalarAttr(builder.getBoolAttr(true));127  }128​129  // 4. 如果需要 Shape Value Inference130  if (need_shape_val_infer) {131    std::vector<std::vector<int64_t>> input_shapes_v;132    for (auto in_op : inputs) {133      if (module::isShape(in_op)) {134        // 读取 shape 张量的值135        auto input_shape_v = module::getShapeTensorValue(in_op);136        input_shapes_v.push_back(input_shape_v);137      } else if (module::isWeight(in_op)) {138        // 读取权重的值（转成 int64）139        auto data = in_op.getDefiningOp<top::WeightOp>().read_as_float();140        std::vector<int64_t> data_v(data->begin(), data->end());141        input_shapes_v.push_back(data_v);142      }143    }144​145    // 用通用方法推导输出 shape 数值146    auto output_shape_v =147        module::commonShapeValInfer(getOperation(), input_shapes_v, out_shape);148    module::bindShapeTensorValue(getOutput(), output_shape_v);149​150    // 如果是标量或一维 shape151    if (out_shape.size() == 1 || out_shape.size() == 0) {152      auto output_shape_v = module::commonShapeValInfer(153          getOperation(), input_shapes_v, out_shape);154      module::bindShapeTensorValue(getOutput(), output_shape_v);155    } else {156      // 形状张量的维度大于 1，发出警告157      dump();158      llvm::errs() << "WARNING: Shape Type Tensor is calculating with a Tensor "159                      "dimension > 1\n";160    }161  }162}c++

文件：`mymlir-opt.cpp`

```


int main(int argc, char **argv) {
  mlir::DialectRegistry registry;
  mlir::MLIRContext context(registry);

  // 注册 Dialect 和 Pass
  registry.insert<MyDialect>();
  registerMyOptimizationPass();

  // 调用 MLIR 提供的主程序处理逻辑（你只需写注册逻辑）
  return mlir::asMainReturnCode(
      mlir::MlirOptMain(argc, argv, "My MLIR Optimizer", registry));
}

```

`MyPass.cpp` 中的实际优化逻辑（C++类中）：

很明显，上面我们写的添加Pass并且执行的代码就在runOnOperation这个函数里

**实际上上面的mian函数则会自动调用它，也就是通过 `mlir::MlirOptMain()` 框架**

```
struct MyOptimizationPass : public PassWrapper<...> {
  void runOnOperation() override {
    mlir::PassManager pm(&getContext());
    pm.addPass(mlir::createInlinerPass());
    pm.addPass(mlir::createCanonicalizerPass());

    if (failed(pm.run(getOperation()))) {
      signalPassFailure(); // 报错处理
    }
  }
};

```



**总体的一个流程如下：**

```
main() 函数
  ↓
调用 MLIR 的 MlirOptMain()
  ↓
根据 -my-opt 命令行参数
  ↓
调用你写的 MyOptimizationPass.runOnOperation()
  ↓
执行 pm.addPass(...) 等优化逻辑

```



### 1.2 自定义一个Pass（TableGen）对于自己定义的Dialect来用

**为什么要自定义Pass呢？**

MLIR本身提供了很多Pass，例如：

- `createCanonicalizerPass()`
- `createInlinerPass()`
- `createCSEPass()`（公共子表达式消除）

但是这些Pass智能处理通用的场景，无法处理自定义Dialect的规则和语义

#### 1.2.1 定义Pass的两种方式  一、 手写C++ Pass

不依赖TableGen知识自己管理和注册，比较麻烦

#### 1.2.1 用TableGen定义 + 自动生成C++框架

作用是：

- 把 Pass 的元信息（名称、作用对象、描述）统一管理；
- 自动生成注册代码、命令行支持；
- 帮你写好继承类的定义，**你只需实现核心逻辑（runOnOperation）**。

**示例：**

```
def MyOptPass : Pass<"my-opt", "func::FuncOp"> {
  let summary = "移除 +0 的无效加法";
  let constructor = "createMyOptPass()";
}
```

在 `.cpp` 里只写：

```
void MyOptPass::runOnOperation() {
  // 你自己的优化逻辑
}
```





## 2、优化的分层学习

**1.1高层优化（初级部分，易入门）**

学会：如何写 PatternRewriter 重写 `my.add %x, 0 → %x`

关键词：自定义 Dialect、Pattern、匹配语义

推荐工具：TableGen + RewritePattern

**1.2 中层优化 **

学会：如何 tile 一个 Linalg 算子、向量化

关键词：Loop nest、SCF、Bufferization、Fusion

推荐工具：Linalg Pass、SCF、PassPipelineRegistration



## 3、 MLIR高层优化

#### 3.1 高层优化实例

**高层优化（High-Level Optimization）是指：**

在保留**程序语义不变**的前提下，**对高抽象层级的 IR**（如算子、函数调用、控制流结构）进行重写、融合、规约等操作，以提升表达清晰度或为后续优化做好准备。

| 特征     | 说明                                       |
| -------- | ------------------------------------------ |
| 优化对象 | 高级 Dialect（如 Toy、MHLO、TOSA、ONNX）   |
| 不涉及   | 硬件、内存细节                             |
| 常用方式 | 重写表达式、融合算子、消除冗余语义         |
| 工具     | RewritePatternSet + PatternRewriter + Pass |

**（1）先定义一个简单的加法操作：**

```
// MyOps.td
def MyAddOp : My_Op<"add", [Commutative]> {
  let arguments = (ins I32:$lhs, I32:$rhs);
  let results = (outs I32:$res);
}

```

**（2）编写Pattern重写规则**

```c++
struct RemoveZeroAddPattern : mlir::OpRewritePattern<MyAddOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(MyAddOp op, PatternRewriter &rewriter) const override {
    if (auto rhs = dyn_cast_or_null<arith::ConstantOp>(op.getRhs().getDefiningOp())) {
      if (rhs.getValue().cast<IntegerAttr>().getInt() == 0) {
        rewriter.replaceOp(op, op.getLhs());
        return success();
      }
    }
    return failure();
  }
};

```

**（3）包装成一个高层优化Pass**

```c++
struct MyHighLevelOptPass : PassWrapper<MyHighLevelOptPass, OperationPass<func::FuncOp>> {
  void runOnOperation() override {
    RewritePatternSet patterns(&getContext());
    patterns.add<RemoveZeroAddPattern>(&getContext());

    if (failed(applyPatternsAndFoldGreedily(getOperation(), std::move(patterns))))
      signalPassFailure();
  }
};
```

注册：

```
std::unique_ptr<Pass> createMyHighLevelOptPass() {
  return std::make_unique<MyHighLevelOptPass>();
}

```

**（4）编写测试用例**

```
func.func @test(%a: i32) -> i32 {
  %c0 = arith.constant 0 : i32
  %x = mydialect.add %a, %c0 : i32
  return %x : i32
}

```

优化之后应该变成：

```
func.func @test(%a: i32) -> i32 {
  return %a : i32
}

```

**（5）推荐练习方向**

**去除加 0、乘 1、减 0、除 1 等冗余操作**

 **匹配多层算子结构，进行融合简化**

 **将 `if x == true then y else y` → `y`（控制流简化）**

 **函数内联（可用系统内建 pass）**



#### 3.2 Pattern重写

就是指自己写一段代码然后识别到对应结构后将其替换

比如定义一个规则：

> 如果我看到 `mydialect.add %x, 0`
>  就替换成 `%x`，因为加 0 是没必要的。

会写一个类，继承自

```
mlir::OpRewritePattern<MyAddOp>
```

然后实现这个函数，通过名字很明显可以看出来其作用：匹配之后重写

```
LogicalResult matchAndRewrite(MyAddOp op, PatternRewriter &rewriter) const

```

| 功能         | 用法                                                     |
| ------------ | -------------------------------------------------------- |
| `match...`   | 看看当前这个 `MyAddOp` 满不满足“加 0”的条件              |
| `rewrite...` | 如果满足，就用 `rewriter` 替换成你想要的 IR（比如 `%x`） |

**带注释的实例：**

```c++
// 声明一个重写规则类，作用在 MyAddOp 上
struct RemoveZeroAddPattern : mlir::OpRewritePattern<MyAddOp> {
  using OpRewritePattern::OpRewritePattern;

  // 这个函数会被自动调用，尝试匹配并重写 Op
  LogicalResult matchAndRewrite(MyAddOp op, PatternRewriter &rewriter) const override {
    // 提取右操作数（rhs）
    Value rhs = op.getRhs();

    // 检查 rhs 是不是一个常量 0
    if (auto constOp = rhs.getDefiningOp<arith::ConstantOp>()) {
      if (constOp.getValue().cast<IntegerAttr>().getInt() == 0) {
        // 如果是加 0，替换整个 MyAddOp 为它的左操作数
        rewriter.replaceOp(op, op.getLhs());
        return success();
      }
    }

    // 否则匹配失败，不做修改
    return failure();
  }
};

```

看到 `x + 0` 就换成 `x`



**这个规则什么时候被执行呢？**

需要写一个 Pass，把这些规则打包起来，然后运行在 IR 上(就是运行在mlir文件上)。

这段代码负责把所有规则装进去并应用：

```c++
RewritePatternSet patterns(&getContext());
patterns.add<RemoveZeroAddPattern>(&getContext());

// 应用所有模式进行优化
applyPatternsAndFoldGreedily(getOperation(), std::move(patterns));

```
