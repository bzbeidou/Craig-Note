# 一、model_transform.py

## 1、model_transform这个同名关键函数内容

这个函数的流程

```
               model_transform(...)
                        ↓
        ┌───────────── init: 设置文件路径
        │
        ├── 调用 self.converter 生成 origin.mlir
        │
        ├── 调用 mlir_opt_for_top 做 Pass 优化，生成最终 mlir
        │
        ├── 检查 pattern 是否按预期应用
        │
        ├── 解析 mlir 模块输入信息
        │
        └── 记录本次转换的文件路径和命令参数

```

## 2、model_validate函数

这是验证原始onnx模型和转换后模型之间的正确性问题，是通过推理验证的

```
1. 加载输入样本 → 构建输入张量 dict
2. 保存成 input.npz 文件
3. 原始模型推理（由子类实现）
4. 如果不做 MLIR 推理，直接返回
5. MLIR 模型推理
6. 对比两个模型的输出 → 报告误差

```



# 二、model_runner.py推理引擎内容

负责接入原始模型（如 ONNX）与 MLIR 模型，进行前后对比推理验证。

**以onnx模型为例理解这个代码**

```
原始模型        MLIR模型
   │              │
   ▼              ▼
[onnx_inference]  [mlir_inference]
   │                        │
   └──→  生成 npz 输出  ←────┘
                │
                ▼
           [对比验证输出]

```



整体内容

| 函数/类名                                            | 类型 | 功能说明                                         | 是否重点（当前阶段） |
| ---------------------------------------------------- | ---- | ------------------------------------------------ | -------------------- |
| `onnx_inference`                                     | 函数 | 运行 ONNX 模型获取参考输出（重点）               | **重点学习**         |
| `mlir_inference`                                     | 函数 | 运行 MLIR 模型并返回输出结果                     | **重点学习**         |
| `final_mlir_inference`                               | 函数 | 内部封装的推理函数（进阶）                       | 可以稍后看           |
| `_mlir_inference_by_cpu``_mlir_inference_by_cuda`    | 函数 | 选择 CPU 或 CUDA 模式跑 MLIR 模型                | 了解即可             |
| `free_mlir_module`                                   | 函数 | 释放推理时加载的 MLIR 模块资源                   | 简单了解             |
| `caffe_inference``tflite_inference``torch_inference` | 函数 | 用于支持其他原始模型类型（Caffe/TFLite/PyTorch） | 可跳过               |
| `model_inference`                                    | 函数 | 封装了对 bmodel/cvimodel 的调用推理              | 后期重点             |
| `_model_inference`                                   | 函数 | 更底层的 bmodel 推理接口                         | 后期了解             |
| `get_cmodel_so``link_cmodel_so``link_custom_so`      | 函数 | 与 C 模型库（.so）动态链接，供后端使用           | 后期了解             |
| `pack_bmodel_context_generator`                      | 函数 | 打包模型上下文（如 chip 类型）用于多线程         | 暂时略过             |
| `get_chip_from_model`                                | 函数 | 识别模型文件是跑在哪种芯片上                     | 暂时了解             |
| `ChipLock`                                           | 类   | 控制推理过程中芯片并发访问锁                     | 暂时略过             |
| `show_fake_cmd`                                      | 函数 | 打印提示信息（伪命令行），用于调试可视化         | 了解即可             |
| `if __name__ == '__main__'`                          | 入口 | 作为命令行执行入口                               | 非重点               |

**目前重点是这三个函数：**

 `onnx_inference(inputs, onnx_path)`

> 用来运行原始 ONNX 模型
>  是 `origin_inference()` 中的核心调用

 `mlir_inference(inputs, mlir_path)`

> 调用 pymlir 的 `run` 函数来推理
>  输出 `.npz` 文件
>  这是用 MLIR 模型进行对比验证的关键步骤

 `final_mlir_inference`（由 `mlir_inference` 内部调用）

> 会选择用 CPU / CUDA 后端运行 MLIR
>  负责调用 pymlir 的实际执行逻辑



如果后面学习了 TPU-MLIR 的 **后端模型生成和部署**，可以进一步深入：

- `model_inference`：用于部署阶段验证 bmodel 的推理效果
- `get_chip_from_model` / `ChipLock`：涉及芯片并发推理安全控制
- `link_cmodel_so`：动态链接编译出来的 C 模型库（如 `model_c.so`）



## 1、onnx_inference函数

整体可以划分为以下几个内容

**（1）生成dump_all_onnx**

如果需要中间层，将中间的激活值 `x.output` 添加到 `model.graph.output` 中，以便可以在 session.run() 时一并输出。

```py
for x in model.graph.node:  # 遍历所有 node
    if x.op_type in no_list:
        continue
    for name in x.output:
        if not name:
            continue
        intermediate_layer_value_info = onnx.helper.ValueInfoProto()
        intermediate_layer_value_info.name = name
        model.graph.output.append(intermediate_layer_value_info)  # 添加为额外输出
        output_keys.append(intermediate_layer_value_info.name + '_' + x.op_type)

```



**（2）准备推理session**

这里有一个先入条件，如果是 `dump_all=True`，就先把 `.onnx` 改造成带中间层输出的新文件，并替换路径。

```py
if dump_all:
    output_keys, onnx_file = generate_onnx_with_all(onnx_file)
```

创建 ONNX 推理 session，默认在 CPU 上执行。

```python
session = onnxruntime.InferenceSession(onnx_file, providers=['CPUExecutionProvider'])
```

**（3）输入准备**

获取onnx输入节点

```python
inodes = session.get_inputs()
only_one = len(inputs) == 1

```

**（5）执行模型推理后返回结果**

```py
outs = session.run(None, data)
```

# 三、mlir_shell.py的内容

**把系统命令 `xxx | mlir-opt | mlir-translate` 转成 Python 函数 `mlir_opt(...)` 来用。**

`mlir_shell.py` 是 `TPU-MLIR` 项目中用于封装各种 **MLIR 命令行工具调用** 的模块，把 MLIR 工具链（比如 `mlir-opt`、`mlir-translate`、`model-deploy` 等）变成 **Python 函数接口**



主要看mlir_opt_for_top()的内容

 `mlir_opt_for_top(...)`

这个是最常用的函数，作用是：使用 `mlir-opt` 工具将 frontend 生成的 `.mlir` 文件优化成用于 `tpu-mlir` 后端的 MLIR。



在TPU-MLIR中将shell命令转成python的方式如下

**1. 第一步，构建shell命令列表**

```
cmd = ["tpuc-opt", mlirfile]
cmd.extend([...])  # 添加各种优化 pass
cmd.extend(["-o", opt_mlirfile])
```

等价于在命令行中写：

```
tpuc-opt xxx_origin.mlir --convert-top-to-tpu -o xxx.mlir

```

只是现在写成了Python的list格式

```
["tpuc-opt", "xxx_origin.mlir", "--convert-top-to-tpu", "-o", "xxx.mlir"]

```

**2.用_os_system()执行它**

```py
_os_system(cmd, log_level=log_level)
```

这里_os_system()的本质就是：

```py
import os
os.system(" ".join(cmd))
```



# 四、mlir_parser.py？

**一句话：把MLIR文件中的内容都进来并解析成结构化数据，抽象成Python类，方便访问和处理**

两个类：Operation和MlirParser

```
MLIR 文件
   ↓
MlirParser 负责解析（读取文件、拆解结构）
   ↓
每个 Operation 行被封装成一个 Operation 对象
   ↓
所有 Operation 对象组成一个有序列表
   ↓
供后续使用：结构分析、输入输出识别、shape提取、类型判断等

```



## 1 `Operation` 类

用来**表示 MLIR 文件中的一个 Operation（Op）**，相当于 IR 中的一个节点。

- 记录 Op 的基本信息：名称、类型、输入输出、属性
- 方便后续按名字、类型、输入输出等方式查找或处理某个 Op

**示例：假设下面这个mlir片段**

```mlir
%0 = "top.Conv"(%input, %weight, %bias) {name = "conv1"} : ...
```

会被解析为一个operation对象，其中会包含类似信息

| 字段     | 示例                             |
| -------- | -------------------------------- |
| name     | `%0`                             |
| op_type  | `"top.Conv"`                     |
| operands | `["%input", "%weight", "%bias"]` |
| attrs    | `{"name": "conv1"}`              |

## 2 `MlirParser` 类

是整个模块的核心类，负责**读取并解析整个 MLIR 文件**，生成对应的 Operation 列表，以及提取模块级别信息。

有以下大致功能

| 功能                           | 说明                                    |
| ------------------------------ | --------------------------------------- |
| `get_input_num()`              | 获取模型的输入数量                      |
| `get_batch_size()`             | 获取输入 batch 大小                     |
| `get_input_op_by_idx(i)`       | 获取第 i 个输入对应的 Operation 信息    |
| `get_op_type_by_op_name(name)` | 根据 op 名字获取它的类型                |
| `get_pre_op_by_op_name(name)`  | 获取某个 Op 的前驱 Op（用于构建依赖图） |
| `get_all_ops()`                | 返回整个 MLIR 中的所有 Operation 对象   |

**示例：**

目前有一个MLIR文件叫做model.mlir，那么可以这样用

```py
parser = MlirParser("model.mlir")
print("模型有几个输入？", parser.get_input_num())
print("第一个输入是哪个Op？", parser.get_input_op_by_idx(0))
print("模型中所有的Op类型：", [op.op_type for op in parser.get_all_ops()])

```



**整个解析过程简单介绍**

(1)`MlirParser` 读入整份 MLIR 文件；

(2)每一条形如 `%x = "xxx.Op"` 的行被识别成一个 Op；

(3)用 `Operation` 类把这些 Op 封装成结构化对象，如：

```
Operation(
    name='%1',
    op_type='top.Conv',
    operands=['%0', '%weight', '%bias'],
    attrs={'name': 'conv1'}
)
```

(4)这些 `Operation` 会被存入 `MlirParser` 的一个列表或字典里，像这样：

```py
self.ops = {
    '%0': Operation(...),
    '%1': Operation(...),
    ...
}

```





**关于MlirParser这个读取mlir的功能分析**

MlirParser可以读入所有的不同dialect的.mlir文件，但是仅仅是进行的文本解读, 而且有一部分还是针对于固定的top这种dailect来处理

也就是说：

- 它能解析出：
	- 每个 Operation 的名字（比如 `%1 = "top.Conv" ...`）、
	- 类型（`top.Conv`, `func.return` 等）、
	- 输入输出名（即 SSA 值）、
	- 形状（如果是 `ShapedType`）、
	- 属性（如 `kernel=[3,3]`）；
- 但它**不会理解这些 Operation 的真正语义**（比如卷积的计算过程、控制流的路径等）；
- 它也不会做**类型检查**或**验证 Dialect 合法性**，这些工作是编译器内部在运行 Pass 时做的。

而且它的有一部分功能是**专门定制过的**，主要针对 `top` Dialect，这部分功能对于其他Dialect肯定是不合适的：

比如下面这些硬编码的判断：明显是针对top Dialect的

```py
if op_type in ["top.Input", "top.None", "func.return"]:
    ...

```





# Python

## 1、自带的vars()函数

`vars()` 是 Python 的内置函数，它返回对象的 `__dict__` 属性（包含对象的所有可写属性）。也就是把对象的属性转换成字典返回

## 2、`解包操作符  --将字典解包为关键字参数

将字典解包为关键字参数(key=value形式)，可以直接传递给函数

```python
class User:
    def configure(self, name, age, email, role="user", is_active=True):
        """配置用户信息"""
        self.name = name
        self.age = age
        self.email = email
        self.role = role
        self.is_active = is_active
        
    def display(self):
        """显示用户信息"""
        print(f"Name: {self.name}")
        print(f"Age: {self.age}")
        print(f"Email: {self.email}")
        print(f"Role: {self.role}")
        print(f"Active: {'Yes' if self.is_active else 'No'}")
        print("-" * 20)

# 创建一个参数对象（模拟argparse的args）
class Args:
    def __init__(self):
        self.name = "Alice"
        self.age = 30
        self.email = "alice@example.com"
        self.role = "admin"  # 覆盖默认值
        # 注意：没有 is_active 参数

# 创建参数对象
args = Args()

print(" 使用 vars() 查看参数对象:")
print(vars(args))  # 转换为字典
# 输出: {'name': 'Alice', 'age': 30, 'email': 'alice@example.com', 'role': 'admin'}

# 创建用户对象
user = User()

print("\n 使用 ** 解包配置用户:")
user.configure(**vars(args))  # 关键步骤：解包字典为关键字参数

print("\n 用户配置结果:")
user.display()
```

**输出：**

```
 使用 vars() 查看参数对象:
{'name': 'Alice', 'age': 30, 'email': 'alice@example.com', 'role': 'admin'}

 使用 ** 解包配置用户:

 用户配置结果:
Name: Alice
Age: 30
Email: alice@example.com
Role: admin
Active: Yes
--------------------
```

**（1）vars（args）做了什么？**

```py
params_dict = vars(args)
# 等效于: 
# params_dict = {
#     'name': 'Alice',
#     'age': 30,
#     'email': 'alice@example.com',
#     'role': 'admin'
# }
```

**（2）`**`做了什么？**

```py
user.configure(**params_dict)
# 完全等价于:
user.configure(
    name='Alice',
    age=30,
    email='alice@example.com',
    role='admin'
)
```

**（3）默认参数如何处理**

- 函数定义中有默认参数 `role="user"` 和 `is_active=True`
- 字典中提供了 `role` → 使用提供的值 `"admin"`
- 字典中没有 `is_active` → 使用默认值 `True`
