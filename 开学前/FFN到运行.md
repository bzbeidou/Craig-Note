#### 1、FFN算子

**Add残差连接**

输入：

(1,128,4096)  (1,128,4096)

输出：

(1,128,4096)



**RMS归一化**

Cast:

输入：

(1,128,4096)

输出：

(1,128,4096)

Pow

输入：

(1,128,4096)

输出：

(1,128,4096)

ReduceMean

输入：

(1,128,4096)

输出：

(1,128)

Add

输入：

(1,128)

输出：

(1,128)

Sqrt

输入：

(1,128)

输出：

(1,128)

Div

输入：

(1,128)

输出：

(1,128)

Mul:

输入：

(1,128)

(1,128,4096)

输出：

(1,128,4096)



SiLU

Cast:

输入

(1,128,4096)

输出：

(1,128,4096)

MatMul

输入

(1,128,4096)

(4096*11008)

输出：

(1,128,11008)

Sigmoid

输入：

(1,128,11008)

输出：

(1,128,11008)

Mul

输入：

(1,128,11008)

(1,128,11008)

输出：

(1,128,11008)



MatMul

输入：

(1,128,4096)

(4096,11008)

输出：

(1,128,11008)



Mul

输入

(1,128,11008)

(1,128,11008)

输出：

(1,128,11008)



MatMul

输入

(1,128,11008)

(11008,4096)

输出：

(1,128,4096)



Add

输入

(1,128,4096)

(1,128,4096)

输出：

(1,128,4096)



#### 2、AI编译器开发流程再解

**AI编译器中的前端工作就是将输入的原始模型转化为中间表示IR**

**中端工作就是把前端输出进一步优化，中端的输出还是模型文件**

**后端工作就是把这个模型文件输出为硬件可执行的指令集**

**Runtime则是将后端生成的硬件可执行指令集加载到芯片中运行**

#### 3、以最简单的Add加法为例展示AI编译器工作

**（1）前端：将模型转化为IR**

首先需要定义一个简单的加法操作的IR。对于我们的例子，IR非常简单，它仅仅表示一个加法操作。

```py
# 前端代码：把输入模型转换为IR
class AddModel:
    def __init__(self, a, b):
        self.a = a
        self.b = b
        
    def to_ir(self):
        # 这里我们将模型转化为一个加法IR
        # IR是一个字典，包含了操作类型和操作数
        return {'op': 'Add', 'inputs': [self.a, self.b]}

# 示例：模型输入
model = AddModel(a=3, b=5)

# 转换为IR
ir = model.to_ir()
print("IR:", ir)

```



```
IR: {'op': 'Add', 'inputs': [3, 5]}
```



```py
# 中端代码：优化IR
def optimize_ir(ir):
    # 如果IR中的加法操作是常数加法，进行优化
    if ir['op'] == 'Add' and isinstance(ir['inputs'][0], int) and isinstance(ir['inputs'][1], int):
        result = ir['inputs'][0] + ir['inputs'][1]
        return {'op': 'Const', 'value': result}
    return ir

# 优化IR
optimized_ir = optimize_ir(ir)
print("Optimized IR:", optimized_ir)
```

```
Optimized IR: {'op': 'Const', 'value': 8}

```

**后端：将IR转化为硬件可执行代码**

```py
# 后端代码：生成硬件执行代码
def generate_backend_code(ir):
    if ir['op'] == 'Add':
        return f"ADD {ir['inputs'][0]}, {ir['inputs'][1]}"
    elif ir['op'] == 'Const':
        return f"LOAD {ir['value']}"
    return ""

# 生成后端代码
backend_code = generate_backend_code(optimized_ir)
print("Backend Code:", backend_code)

```



```
Backend Code: LOAD 8
```

**Runtime：加载并执行代码**

示例中，Runtime只需要执行生成的加法操作。实际上，Runtime会更复杂，它负责管理整个计算图的执行、调度任务、加载数据等。

```py
# Runtime代码：执行硬件指令
def runtime_execute(backend_code):
    print(f"Executing on hardware: {backend_code}")

# 在Runtime中执行
runtime_execute(backend_code)
```

输出：

```
Executing on hardware: LOAD 8
```

