

 **AbsOp** 的接口：

**getFLOPs()** 

**init() / deinit()** 

**inference()** 

**shape_inference()** 

```c++

int64_t top::AbsOp::getFLOPs() { 
  // 返回算子的浮点运算次数
  return module::getNumElements(getOutput()); 
}
```
```c++
// ---------------------------
// 2. init()
// ---------------------------
LogicalResult top::AbsOp::init(InferenceParameter &p) { 

  return success(); 
}

// ---------------------------
// 3. deinit()
// ---------------------------
void top::AbsOp::deinit(InferenceParameter &p) {
  // 推理完成后的资源释放
  // 这里没有额外资源
}

// ---------------------------
// 4. inference()
// ---------------------------
LogicalResult top::AbsOp::inference(InferenceParameter &p) {
  // 运行推理,计算输出值的地方
  auto num_element = module::getNumElements(getOutput()); // 获取输出元素总数

#pragma omp parallel for schedule(static, omp_schedule(num_element))
  for (int i = 0; i < num_element; ++i) {
    auto val = p.inputs[0][i];        
    p.outputs[0][i] = std::abs(val);  
  }

  return success(); // 推理成功
}

// ---------------------------
// 5. shape_inference()
// ---------------------------
void top::AbsOp::shape_inference() { 
  // 推导输出 shape
  // 对于 Abs，输出 shape 和输入 shape 相同
  common_shape_inference(getOperation()); 
}

```



### add

```c++
// =======================
// 1. 计算 FLOPs
// =======================
int64_t top::AddOp::getFLOPs() {
  // 总 FLOPs = 元素数 × (输入数量 - 1)
  // 例如两个输入：每个元素做一次加法
  // 如果有 ReLU，再加一次激活操作
  return module::getNumElements(getOutput()) *
         (getInputs().size() - 1 + (getDoRelu() ? 1 : 0));
}

// =======================
// 2. 初始化
// =======================
LogicalResult top::AddOp::init(InferenceParameter &p) {
  // 如果是两个输入，可以走 DNNL Binary 算子路径
  if (getInputs().size() == 2) {
    auto binary = new Binary();  // 创建一个 DNNL Binary 加法对象
    
    // 注意：这里原本有 Binary 对象的参数配置代码（被注释掉了）
    // 实际运行时会在 inference() 中再配置一次

    p.handle = (void *)binary; // 保存指针到 handle，供推理时使用
  } else {
    // 多输入加法走手写路径，不用 DNNL
    p.handle = nullptr;
  }
  return success();
}

// =======================
// 3. 资源释放
// =======================
void top::AddOp::deinit(InferenceParameter &p) {
  if (p.handle != nullptr) {
    auto binary = (Binary *)p.handle;
    delete binary;     // 释放 Binary 对象
    p.handle = nullptr;
  }
}

// =======================
// 4. 推理实现
// =======================
LogicalResult top::AddOp::inference(InferenceParameter &p) {
  // 推导广播后的输出 shape，并更新到 output
  auto output_shape = computer_broadcast_shape(getOperation());
  module::setShape(getOutput(), output_shape);

  if (getInputs().size() == 2) {
    // 双输入路径 —— 用 DNNL Binary 加速
    if (p.handle == nullptr) {
      return failure(); // 初始化时没创建 Binary，直接报错
    }

    auto binary = (Binary *)p.handle;

    auto lhs_shape = module::getShape(getInputs()[0]);
    auto rhs_shape = module::getShape(getInputs()[1]);

    // 配置 Binary 算子
    (*binary)
        .hs(p.inputs[0], p.inputs[1], lhs_shape, rhs_shape)  // 设置左右输入
        .dst(p.outputs[0], module::getShape(getOutput()))    // 设置输出缓冲区
        .do_relu(getDoRelu())                                // 是否做 ReLU
        .relu_limit(getReluLimit().convertToDouble())        // ReLU 上限
        .algorithem(algorithm::binary_add)                   // 算法类型 = 加法
        .setup();                                            // 完成配置

    binary->run(); // 执行加法运算

  } else {
    // 多输入路径 —— 手写并行加法
    auto num_elem = module::getNumElements(getOutput());
#pragma omp parallel for schedule(static, omp_schedule(num_elem))
    for (int64_t i = 0; i < num_elem; i++) {
      p.outputs[0][i] = 0;
      for (auto in : p.inputs) {
        if (in != nullptr) {
          p.outputs[0][i] += in[i];
        }
      }
    }
    // 如果开启了 ReLU
    if (getDoRelu()) {
      auto limit = getReluLimit().convertToDouble();
      function_relu(p.outputs[0], p.outputs[0], num_elem, limit);
    }
  }

  return success();
}

// =======================
// 5. 形状推导
// =======================
void top::AddOp::shape_inference() {
  // 1. 广播推导基础 shape
  broadcast_shape_inference(getOperation());
  bool has_scalar = false;
  for (int i = 0; i < getNumOperands(); i++) {
    auto value = getInputs()[i];
    broadcast_tensor_reshape(getOutput(), value); // 对标量或广播输入 reshape
    has_scalar = has_scalar && module::isScalar(value.getDefiningOp());
  }

  // 2. 判断是否需要 Shape Value Inference（即推导 shape 张量的数值）
  auto inputs = getInputs();
  bool need_shape_val_infer =
      std::all_of(inputs.begin(), inputs.end(),
                  [](auto in_op) {
                    return module::isWeight(in_op) || module::isShape(in_op);
                  }) &&
      std::any_of(inputs.begin(), inputs.end(),
                  [](auto in_op) { return module::isShape(in_op); });
  
  need_shape_val_infer &=
      std::any_of(inputs.begin(), inputs.end(),
                  [](auto in_op) { return module::isShape(in_op); });

  // 3. 标记输出是否为标量
  auto out_shape = module::getShape(getOutput());
  if (out_shape.size() == 1 && out_shape[0] == 1 && has_scalar) {
    auto context = getContext();
    mlir::Builder builder(context);
    setIsScalarAttr(builder.getBoolAttr(true));
  }

  // 4. 如果需要 Shape Value Inference
  if (need_shape_val_infer) {
    std::vector<std::vector<int64_t>> input_shapes_v;
    for (auto in_op : inputs) {
      if (module::isShape(in_op)) {
        // 读取 shape 张量的值
        auto input_shape_v = module::getShapeTensorValue(in_op);
        input_shapes_v.push_back(input_shape_v);
      } else if (module::isWeight(in_op)) {
        // 读取权重的值（转成 int64）
        auto data = in_op.getDefiningOp<top::WeightOp>().read_as_float();
        std::vector<int64_t> data_v(data->begin(), data->end());
        input_shapes_v.push_back(data_v);
      }
    }

    // 用通用方法推导输出 shape 数值
    auto output_shape_v =
        module::commonShapeValInfer(getOperation(), input_shapes_v, out_shape);
    module::bindShapeTensorValue(getOutput(), output_shape_v);

    // 如果是标量或一维 shape
    if (out_shape.size() == 1 || out_shape.size() == 0) {
      auto output_shape_v = module::commonShapeValInfer(
          getOperation(), input_shapes_v, out_shape);
      module::bindShapeTensorValue(getOutput(), output_shape_v);
    } else {
      // 形状张量的维度大于 1，发出警告
      dump();
      llvm::errs() << "WARNING: Shape Type Tensor is calculating with a Tensor "
                      "dimension > 1\n";
    }
  }
}
```