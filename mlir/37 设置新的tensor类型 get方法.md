

**构造一个新的张量类型，比如tensor<1x2x3xf32>**

#### 1 `RankedTensorType`的`get`方法

比如tensor<1x2x3xf32> `mlir::TensorType`

| 类名                       | 举例                      |
| -------------------------- | ------------------------- |
| `mlir::RankedTensorType`   | `tensor<1x3x224x224xf32>` |
| 同上，部分维度是 `?`       | `tensor<?x3x224x224xf32>` |
| `mlir::UnrankedTensorType` | `tensor<*xf32>`           |

`RankedTensorType::get(...)` 

用来构造一个新的 `RankedTensorType` 对象

原型如下，有两个输入

```c++
static RankedTensorType get(ArrayRef<int64_t> shape, Type elementType);
```

- `shape`: 张量的维度
- `elementType`: 每个元素的类型

返回值是一个 `RankedTensorType` 对象

#### 2  `builder.getF32Type()` 

用于快速创建常见的基本类型，可以获取数据类型

| 方法                               | 返回的 Type     |
| ---------------------------------- | --------------- |
| `builder.getF32Type()`             | 32 位浮点类型   |
| `builder.getF16Type()`             | 半精度浮点      |
| `builder.getBF16Type()`            | bfloat16        |
| `builder.getI8Type()`              | 8 位整数        |
| `builder.getI32Type()`             | 32 位整数       |
| `builder.getF64Type()`             | 64 位浮点       |
| `builder.getIntegerType(8)`        | 自定义位宽整数  |
| `builder.getIntegerType(8, false)` | 无符号 8 位整数 |
| `builder.getIndexType()`           | 索引类型        |

#### 3、举例说明应用

```c++
OpBuilder builder(ctx);
auto i32 = builder.getI32Type();

auto tensor_i8  = RankedTensorType::get({1, 3, 224, 224}, i8);

```

#### 4 get

```c++
auto f32_1 = builder.getF32Type();
auto f32_2 = builder.getF32Type();

```

```c++
assert(f32_1 == f32_2);  
```

MLIR 中的所有 `Type`、`Attribute`、`Location` 等对象多了也没用。

当你调用 `RankedTensorType::get({1,3,224,224}, f32)` 时，会检查当前是不是已经有这个tensor类型的张量了，如果有的话直接返回，没有的话就创建。

同样的builder.get这个方法也是，但是一般这个情况都是获取已经定义好的，因为我们想用的数据类型在td文件里已经定义，或者mlir本身就提供了。



