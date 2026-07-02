# `OnnxConverter`

#### 1. **`translate_onnx(key, val)`**

将 ONNX 操作符的属性（`key` 和 `val`）转换为适合在当前转换过程中使用的格式。

- **`key`**：ONNX 操作的属性名称，例如 `axis`、`dtype` 等。
- **`val`**：是属性的值，通常是字符串、数字、列表等类型。

**简化的示例**：

```
def translate_onnx(key, val):
    if key == "axis":
        return int(val)
    elif key == "dtype":
        return onnx_dtype(val)
    # 可以继续根据其他属性进行处理
```

#### 2. **`onnx_dtype(dtype)`**

- 将 `float32` 转换为 `np.float32`
- 将 `int64` 转换为 `np.int64`

```py
def onnx_dtype(dtype):
    # 数字类型
    if isinstance(dtype, Number):
        onnx_dtype = dtype
    # 字符串
    elif isinstance(dtype, str):
        onnx_dtype = onnx.TensorProto.DataType.Value(dtype)
    else:
        raise RuntimeError("dtype should be number or str.")
    return mapping.TENSOR_TYPE_TO_NP_TYPE[onnx_dtype]

```



`mapping.TENSOR_TYPE_TO_NP_TYPE` 将 ONNX 数据类型映射到对应的 NumPy 类型。根据 `onnx_dtype`，通过字典查找得到相应的 NumPy 数据类型并返回。

**举例：**

```py
mapping.TENSOR_TYPE_TO_NP_TYPE = {
    onnx.TensorProto.DataType.FLOAT: np.float32,
    onnx.TensorProto.DataType.INT64: np.int64,
    onnx.TensorProto.DataType.FLOAT16: np.float16,
    # 更多的映射
}

# 假设调用的 onnx_dtype 函数
dtype = "float32"
numpy_type = onnx_dtype(dtype)

print(numpy_type)  # 输出: <class 'numpy.float32'>

```

如果 `dtype` 是 `"float32"`，`onnx.TensorProto.DataType.Value(dtype)` 会返回 `onnx.TensorProto.DataType.FLOAT`。

然后，`mapping.TENSOR_TYPE_TO_NP_TYPE[onnx_dtype]` 会返回 `np.float32`。



#### 3. **`convert_onnx_attribute_proto(attr_proto)`**

将 ONNX 模型中的属性原型（`attr_proto`）转换为适合的 Python 对象或者数据类型。这通常是在处理 ONNX 的 `AttributeProto` 类型时使用的。

**简化示例**：

```
def convert_onnx_attribute_proto(attr_proto):
    # 假设 attr_proto 是某个属性的 proto 类型
    if attr_proto.HasField('i'):  # 如果是整数
        return attr_proto.i
    elif attr_proto.HasField('f'):  # 如果是浮点数
        return attr_proto.f
    elif attr_proto.HasField('strings'):  # 如果是字符串
        return attr_proto.strings
    # 可以继续处理其他类型
```



#### 1. **`BaseNode`**



```py
class BaseNode:
    def __init__(self, info):
        self.name = str(info["name"])
        self.op_type = str(info["op_type"])
        self.attrs = dict(info["attrs"])
        self.inputs = list(info["inputs"])
        self.outputs = list(info["outputs"])
        self.shape_info = dict()

# 示例：加法节点
add_node_info = {
    "name": "add1",
    "op_type": "Add",
    "attrs": {"axis": 0},
    "inputs": ["input1", "input2"],
    "outputs": ["output_add"]
}

# 示例：乘法节点
mul_node_info = {
    "name": "mul1",
    "op_type": "Multiply",
    "attrs": {"scalar": 2},
    "inputs": ["output_add"],
    "outputs": ["output_mul"]
}

# 示例：ReLU 节点
relu_node_info = {
    "name": "relu1",
    "op_type": "ReLU",
    "attrs": {},
    "inputs": ["output_mul"],
    "outputs": ["output_relu"]
}

# 创建 BaseNode 对象
add_node = BaseNode(add_node_info)
mul_node = BaseNode(mul_node_info)
relu_node = BaseNode(relu_node_info)

# 打印每个节点的信息
print(f"Node Name: {add_node.name}, Operation Type: {add_node.op_type}, Inputs: {add_node.inputs}, Outputs: {add_node.outputs}")
print(f"Node Name: {mul_node.name}, Operation Type: {mul_node.op_type}, Inputs: {mul_node.inputs}, Outputs: {mul_node.outputs}")
print(f"Node Name: {relu_node.name}, Operation Type: {relu_node.op_type}, Inputs: {relu_node.inputs}, Outputs: {relu_node.outputs}")

```

#### 2. **`OnnxNode(BaseNode)`**

```py
class OnnxNode(BaseNode):

    def __init__(self, node):
        info = dict()
        info["name"] = node.output[0] # 第一个输出张量的名称
        info["op_type"] = node.op_type # 操作类型
        info["attrs"] = [(attr.name, translate_onnx(attr.name, convert_onnx_attribute_proto(attr)))
                         for attr in node.attribute]
        info["inputs"] = node.input # 输入
        info["outputs"] = node.output # 输出
        super().__init__(info) #调用父类的构造函数
        self.node_proto = node #存储原始的node信息
```

**`translate_onnx(attr.name, convert_onnx_attribute_proto(attr))`**：这是对每个属性值进行转换：

**`convert_onnx_attribute_proto(attr)`**：将属性的 `proto` 格式转换为 Python 对象。

**`translate_onnx(attr.name, ...)`**：将转换后的属性值进一步处理，转换为适当的类型。

```py
# 假设我们有一个 ONNX 加法节点
class OnnxAddNode:
    def __init__(self):
        self.op_type = "Add"
        self.output = ["output_add"]
        self.input = ["input1", "input2"]
        self.attribute = []  

# 创建一个简单的 ONNX Add 节点实例
onnx_add_node = OnnxAddNode()

# 将其传递给 OnnxNode
add_node = OnnxNode(onnx_add_node)

# 打印出转换后的 BaseNode 属性
print(f"Node Name: {add_node.name}")
print(f"Operation Type: {add_node.op_type}")
print(f"Inputs: {add_node.inputs}")
print(f"Outputs: {add_node.outputs}")
```

```py
# 输出
Node Name: output_add
Operation Type: Add
Inputs: ['input1', 'input2']
Outputs: ['output_add']
```

此外通过保存原始的 `node_proto`，可以在后续处理中访问 ONNX 节点的所有详细信息。



#### 3. **`OnnxConverter(BaseConverter)`**

```
class OnnxConverter(BaseConverter):
    def __init__(self, onnx_model):
        super().__init__(no_save=False)
        self.onnx_model = onnx_model  # ONNX 模型

    def generate_mlir(self, mlir_file):
        for node in self.onnx_model.graph.node:
            onnx_node = OnnxNode(node.name, node.input, node.output, node.op_type)
            # 将 ONNX 节点转换为 MLIR 操作
            # 保存到 mlir 文件中
```



# Python

## 1. lambda关键字  是python的关键字

`lambda` 是 Python 中的一个关键字，用于定义匿名函数。



**匿名函数就是没名字的函数，通常用案例进行短小的操作**



**语法格式：**

```py
lambda argument_list: expression
```

`argument_list`：是一个或多个输入参数。

`expression`：是一个返回值，它会计算并返回给调用者。



**举例**

```py
lambda x: int(x)
```

`lambda x: int(x)` 就是定义了一个匿名函数，这个函数接受一个参数 `x`，然后将 `x` 转换成整数并返回。



**lambda与普通函数的对比**

**普通函数**

```py
def convert_to_int(x):
    return int(x)
```

**`lambda`函数定义**

```py
convert_to_int = lambda x: int(x)
```



**`lambda` 函数在字典中的应用**

```py
onnx_attr_translator = {
    "axis": lambda x: int(x),
    "axes": lambda x: [int(a) for a in x],
    "dtype": lambda x: onnx_dtype(x),
    "keepdims": lambda x: bool(x),
    "to": lambda x: onnx_dtype(x),
}
```

`lambda` 函数作为字典的值，用来处理不同的 ONNX 操作属性。每个键（如 `"axis"`, `"axes"`, `"dtype"`, etc.）对应一个 `lambda` 函数，用于转换该属性的值。

**具体使用**

```py
# 假设属性值为 '3.14'，需要将其转换为整数
axis_value = "3.14"
axis_transformed = onnx_attr_translator["axis"](axis_value)
print(axis_transformed)  # 输出: 3
```

通过 `onnx_attr_translator["axis"](axis_value)`，我们使用字典中对应的 `lambda` 函数，将 `axis_value` 转换为整数。





## 2. 字典的get方法

```py
def translate_onnx(key, val):
    return onnx_attr_translator.get(key, lambda x: x)(val)
```

**`get()` 方法** 是字典的一个内置方法，它用来获取指定键（`key`）对应的值。如果 `key` 存在，则返回该值；如果 `key` 不存在，则返回第二个参数（在这个例子中是 `lambda x: x`）。



这里第二个参数是一个函数，所以返回的东西就相当于一个函数名，比如

```py
func = lambda x : x;
# 也就是说lambda x : x;就是相当于这个func是一个函数名，想要用这个函数肯定需要参数
# 也就是这样用func();
# 所以后面的(val)就是输入的参数
```



**除了上面说的这个匿名函数，get方法在普通函数也可以用**

```py
# 定义一些简单的函数
def add(x, y):
    return x + y

def multiply(x, y):
    return x * y

# 将函数存储到字典中
operation_dict = {
    "add": add,
    "multiply": multiply
}

# 通过 get() 方法取出并调用函数
operation = operation_dict.get("add")  # 通过 "add" 获取 add 函数
result = operation(2, 3)  # 调用 add 函数，传入参数 2 和 3
print(result)  # 输出: 5

operation = operation_dict.get("multiply")  # 通过 "multiply" 获取 multiply 函数
result = operation(2, 3)  # 调用 multiply 函数，传入参数 2 和 3
print(result)  # 输出: 6

```
