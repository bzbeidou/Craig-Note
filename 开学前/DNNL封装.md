## Binary.cpp 专门用作二元计算的工具类（这是自己封装的）

这里在lib/Support/Dnnl/Binary.cpp之下，则是因为本质上还是用了dnnl的内容。

| 特性               | 说明                                    |
| ------------------ | --------------------------------------- |
| 高性能             | 针对多种 CPU 架构 SIMD 指令优化         |
| 支持多种数据类型   | float32, bf16, int8 等                  |
| 支持多种算子       | Binary（加、减、乘、除）、卷积、池化等  |
| 与深度学习框架兼容 | 可作为 PyTorch、TensorFlow 的底层计算库 |

*

(1)engine 引擎  表示计算在那个硬件上跑

```c++
dnnl::engine eng(dnnl::engine::kind::cpu, 0);
```

`engine::kind::cpu` → 跑在 CPU 上

`0` → 设备编号（通常 CPU 只有 0）

(2) stream   表示执行上下文（执行指令的通道）,后面所有算子执行都要放到这个 `stream` 上

```c++
dnnl::stream eng_stream(eng);
```

(3)memory（内存/Tensor）  存放数据的容器

创建这个memory对象的时候需要传入这个memory的描述，这个描述里包括形状、数据类型等

其实这里的memory就是存放张量用的

```c++
dnnl::memory::dims shape = {2, 3};

auto mem_desc = dnnl::memory::desc(shape, dnnl::memory::data_type::f32,
                                   dnnl::memory::format_tag::nc);
dnnl::memory mem(mem_desc, eng);
float *ptr = static_cast<float*>(mem.get_data_handle());
ptr[0] = 1.0f;
```

（4）primitive

```c++
using tag = dnnl::memory::format_tag;
using dt = dnnl::memory::data_type;

// 输入 shape
dnnl::memory::dims shape = {2, 3};

// memory desc
auto mem_desc = dnnl::memory::desc(shape, dt::f32, tag::nc);

// 输入、输出
dnnl::memory lhs(mem_desc, eng);
dnnl::memory rhs(mem_desc, eng);
dnnl::memory dst(mem_desc, eng);

// 算子描述符
auto binary_d = dnnl::binary::desc(
    dnnl::algorithm::binary_add, mem_desc, mem_desc, mem_desc);

// primitive 描述
auto binary_pd = dnnl::binary::primitive_desc(binary_d, eng);

// 创建 primitive（算子对象）
dnnl::binary binary_prim(binary_pd);

```

（5）执行算子

```c++
binary_prim.execute(eng_stream,
    {{DNNL_ARG_SRC_0, lhs},
     {DNNL_ARG_SRC_1, rhs},
     {DNNL_ARG_DST, dst}});
eng_stream.wait();

```

```c++
#include <iostream>
#include <vector>
#include "dnnl.hpp"

using namespace dnnl;

int main() {
    engine eng(engine::kind::cpu, 0);
    stream eng_stream(eng);

    memory::dims shape = {2, 3};
    auto md = memory::desc(shape, memory::data_type::f32, memory::format_tag::nc);

    memory lhs(md, eng);
    memory rhs(md, eng);
    memory dst(md, eng);

    // 写入输入数据
    float* lhs_data = static_cast<float*>(lhs.get_data_handle());
    float* rhs_data = static_cast<float*>(rhs.get_data_handle());
    for (int i = 0; i < 6; i++) {
        lhs_data[i] = i;
        rhs_data[i] = i * 10;
    }

    // 创建 binary add primitive
    auto binary_d = binary::desc(algorithm::binary_add, md, md, md);
    auto binary_pd = binary::primitive_desc(binary_d, eng);
    auto binary_prim = binary(binary_pd);

    // 执行
    binary_prim.execute(eng_stream,
        {{DNNL_ARG_SRC_0, lhs},
         {DNNL_ARG_SRC_1, rhs},
         {DNNL_ARG_DST, dst}});
    eng_stream.wait();

    // 输出结果
    float* dst_data = static_cast<float*>(dst.get_data_handle());
    for (int i = 0; i < 6; i++) {
        std::cout << dst_data[i] << " ";
    }
    std::cout << std::endl;
}

```



### 1.1 如果不封装 如下  还是挺复杂的

得自己写 **engine、stream、memory desc、primitive desc** 等等很多代码,这里很麻烦。

```c++
#include <dnnl.hpp>
#include <vector>
using namespace dnnl;

int main() {
    // 1. 定义 engine 和 stream
    engine eng(engine::kind::cpu, 0);
    stream eng_stream(eng);
    
    // 2. 定义输入数据
    std::vector<float> a = {1, 2, 3, 4};
    std::vector<float> b = {10, 20, 30, 40};
    std::vector<float> c(4);
    
    // 3. 定义 memory 描述符和 memory 对象
    memory::dims dims = {4};
    auto md = memory::desc(dims, memory::data_type::f32, memory::format_tag::x);

    memory mem_a(md, eng);
    memory mem_b(md, eng);
    memory mem_c(md, eng);

    // 把数据拷贝进去
    write_to_dnnl_memory(a.data(), mem_a);
    write_to_dnnl_memory(b.data(), mem_b);

    // 4. 定义 primitive 描述符（binary: Add）
    binary::primitive_desc pd(
        eng, algorithm::binary_add, mem_a.get_desc(), mem_b.get_desc(), mem_c.get_desc()
    );
    
    // 5. 创建 primitive 并执行
    binary bin_op(pd);
    bin_op.execute(eng_stream, {
        {DNNL_ARG_SRC_0, mem_a},
        {DNNL_ARG_SRC_1, mem_b},
        {DNNL_ARG_DST,   mem_c}
    });
    eng_stream.wait();

    // 6. 取结果
    read_from_dnnl_memory(c.data(), mem_c);
}
```

### 1.2封装直接用就行

```c++
#include "Binary.h"  // 封装好的类

int main() {
    // 输入数据
    std::vector<float> a = {1, 2, 3, 4};
    std::vector<float> b = {10, 20, 30, 40};
    std::vector<float> c(4);

    // 1. 创建算子对象
    Binary add_op;

    // 2. 设置输入输出 shape + 数据
    add_op.setInput(a, {4});
    add_op.setInput(b, {4});
    add_op.setOutput(c, {4});

    // 3. 初始化算子
    add_op.setup();

    // 4. 执行
    add_op.run();

    // 5. 输出结果
    // c = {11, 22, 33, 44}
}

```

# c++

## 1、memset函数  来自头文件`<cstring>`

用来把一块内存区域全部设置成一个值，常用把一片区域全部设置成0

**函数原型：简化的**

```c++
void* memset(void* ptr, int value, size_t num_bytes);
```

`ptr`：内存起始地址

`value`：要写入的**字节值**，比如要写入0，会被转换成 `unsigned char`

`num_bytes`：要写多少个**字节**

第一个参数是地址，从这个地址开始，后面的numbytes这些字节数，全部写入value这个值



**例子**

```c++
auto num_elem = module::getNumElements(getOutput()); // 输出张量一共有多少个元素
auto out_type = module::getStorageType(getOutput()); // 输出的存储数据类型（例如 F32/INT8 等）

memset(p.outputs[0], 0, num_elem * sizeof(float));
```

这里就是把输出buffer初始化为0，这里的buffer就是输出所占的内存

## 2、c++的OpenMP  Open Multi-Processing

就是用来帮助加快循环运行的，去掉完全没影响，使用这个循环就可以并行运行，通过cpu的多核同时执行任务

### 2.1 语法

你在循环前加一行 `#pragma omp parallel for`

编译器会生成多线程代码：

- 创建多个线程（线程数通常等于 CPU 核心数）
- 自动把循环迭代拆分给每个线程
- 循环结束后，线程汇合（join），保证结果正确

**这里的前提是循环里的内容不是互相影响的，否则不能划分**

```c++
#include <omp.h>
#include <iostream>

int main() {
    int N = 10;
    int arr[10];

    #pragma omp parallel for
    for (int i = 0; i < N; i++) {
        arr[i] = i * i;
        // 多个线程同时执行不同的 i
    }

    for (int i = 0; i < N; i++) {
        std::cout << arr[i] << " ";
    }
}

```
