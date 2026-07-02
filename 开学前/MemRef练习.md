## 一、手写练习

#### 1、手动创建一个memref，向里面store，再从里面load

**（1）什么是memref(Memory Reference)**

一块可以读写的连续内存区域

mlir大致做了以下的事情

```
1. 创建一块能装 4 个整数的内存；
2. 把 42 写入第 2 个位置；
3. 再把那个位置的值读取出来；
4. 返回读取到的值。
```

**（2）手写MLIR**

```mlir
module {
  "func.func"() ({
    ^entry:
      // 常量定义
      %c4 = "arith.constant"() {value = 4 : index} : () -> index
      %c2 = "arith.constant"() {value = 2 : index} : () -> index
      %val = "arith.constant"() {value = 42 : i32} : () -> i32

      // 创建一个 memref<4xi32> 的缓冲区
      %buffer = "memref.alloc"() : () -> memref<4xi32>

      // store val 到 buffer[2]
      "memref.store"(%val, %buffer, %c2) : (i32, memref<4xi32>, index) -> ()

      // load buffer[2]
      %loaded = "memref.load"(%buffer, %c2) : (memref<4xi32>, index) -> i32

      // 返回
      "func.return"(%loaded) : (i32) -> ()
  }) {sym_name = "memref_example", function_type = () -> i32} : () -> ()
}

```

## 二、MLIR

#### 1、memref是什么？

`memref` 是 MLIR 中表示 **可读写的内存块（数组、缓冲区）** 的一种机制。

**（1）memref是一个dialect**

这个dialect提供一套标准的**操作Op**和**类型Type**来操作内存块，一定注意提供了操作和类型两种东西。非常类似于c++中数组，也是可以开辟内存和管理

**理解了其中包含Op和Type后面就很好学了**



**（2）常见memref类型Type**

下面这四个都是在这个memref的dialect里定义的类型语法

| 类型语法          | 含义                        |
| ----------------- | --------------------------- |
| `memref<4xi32>`   | 固定大小，长度为4的一维数组 |
| `memref<?xi32>`   | 动态大小的一维数组          |
| `memref<2x4xf32>` | 二维数组，2行4列浮点数      |
| `memref<*xi32>`   | 不定形状（任意大小）        |



**（3）这个dialect里的典型操作Op**

| 操作               | 含义         |
| ------------------ | ------------ |
| `"memref.alloc"`   | 分配内存     |
| `"memref.dealloc"` | 释放内存     |
| `"memref.load"`    | 从地址读取值 |
| `"memref.store"`   | 写入值到地址 |
| `"memref.copy"`    | 拷贝内存内容 |



**（4）简单了解开辟空间**

在mlir里的这句话：

```
%buffer = memref.alloc() : memref<4xi32>
```

并不会真正真正在内存开辟空间，这只是一个IR，还要继续往底层才会开辟空间



其实就是和c++这种代码一样的

```c++
int* p = new int[10];
```

这只是一个说明，具体分配内存空间还需要经过编译到底层才可以



#### 2、内建Dialect和memref等dialect的注册机制

MLIR 中有一个特殊的 Dialect，叫做 `BuiltinDialect`，它包含了一些基本的元素：

- `i32`, `f32`, `index` 等基础类型
- `tensor<...>`，虽然也看起来像是 memref 但其实是 Tensor 类型
- `func.func`, `module` 等

**这些 Dialect 是 MLIR 启动时自动注册的**，不需要手动插入。



但是向memref这种dialect以及自己定义的dialect则需要显示注册

```
registry.insert<mlir::memref::MemRefDialect>();
```

注册之后就能识别到.mlir文件里用到的其他dialect的Op和Type等等



#### 3、c++里的注册和.mlir文件是怎么连接在一起的呢？

手写了如下MLIR文件:

```
%a = memref.alloc() : memref<4xi32>
```

你这个 `.mlir` 文件要运行、要生效，就得加载进去某个工具，例如：

```
mlir-opt my.mlir
```

**所以需要在mlir-opt里去注册用到的Dialect**



**4、 mlir-opt工具 ，注册怎么涉及到这个工具呢？**

`mlir-opt` 是什么？是不是官方工具？我们能不能改？

是的，`mlir-opt` 是 **MLIR 官方提供的工具**。

它的作用是：

- 接收 `.mlir` 文件
- 注册各种 Dialect
- 应用 Pass
- 输出变换后的 IR

你可以把它类比为 **“MLIR 版的 gcc/clang”**，专门用于做中间表示（IR）优化。



**关键问题：自己写的 Dialect 和注册，怎么和 `mlir-opt` 扯上关系？**

**方法一：你修改源码中的 `mlir-opt` 主函数，注册自己的 Dialect**

官方源码中有 `mlir-opt` 的 main 函数，一般在：llvm-project/mlir/tools/mlir-opt/mlir-opt.cpp

这里修改这个文件的内容，也就是添加上自己注册的部分

```c++
int main(int argc, char **argv) {
  mlir::DialectRegistry registry;
  registry.insert<mlir::arith::ArithDialect,
                  mlir::memref::MemRefDialect,
                  mlir::toy::ToyDialect>(); // 你自己加的

  mlir::MLIRContext context(registry);
  ...
}
```

再用cmake重新编译以下得到可执行文件mlir-opt即可

**方法二：**

也就是复制一份mlir-opt的源码，然后改个名字叫做my-opt等等就可以，然后加上自己注册的部分即可，这样不会污染官方库

`mlir-opt` 的源码中确实引用了很多 MLIR 官方代码吗？

```
#include "mlir/IR/DialectRegistry.h"
#include "mlir/InitAllDialects.h"
#include "mlir/InitAllPasses.h"
#include "mlir/Tools/mlir-opt/MlirOptMain.h"
```

Dialect、Pass、Parser、IR、ExecutionEngine 

**使用mlir官方准备的外部项目模板（Standalone 项目）**

官方在 MLIR 的源码里已经为你准备好了一个“完全可以自己扩展的外部项目模板”，支持：

- 加载自己的 Dialect
- 写自己的 `opt` 工具
- 使用原始 MLIR 的所有头文件和库
- 支持 CMake 构建系统自动集成 MLIR/LLVM

```
my-mlir-project/
├── CMakeLists.txt        ← 配置整个工程用的
├── cmake/
│   └── FindMLIR.cmake    ← 官方提供的脚本
├── lib/
│   └── MyDialect/
│       ├── MyDialect.cpp
│       ├── MyDialect.h
│       └── MyDialect.td
├── tools/
│   └── my-opt/
│       └── my-opt.cpp    ← 类似 mlir-opt 的自定义工具
└── test/
    └── simple.mlir
```

这样只需要构建就能使用mlir里的全部功能以及自己定义的功能

```
mkdir build && cd build
cmake .. -G Ninja \
  -DMLIR_DIR=/path/to/llvm-project/build/lib/cmake/mlir
ninja
```



#### 5、模板内容解析  

llvm-project/mlir/examples/standalone/

```
include/
lib/
standalone-opt/
standalone-plugin/
standalone-translate/
test/
```

**为了演示多个“工具场景”**（如：`opt`、`translate`、`plugin`），所以把不同工具的 `main.cpp` 放在了不同文件夹中（比如 `standalone-opt/`），我们如果只要写自己的mlir-opt那么就直接改这个文件夹下的内容就行

| 文件夹 / 文件           | 作用                                                 |
| ----------------------- | ---------------------------------------------------- |
| `include/`              | 存放 Dialect 的头文件（如 `StandaloneDialect.h`）    |
| `lib/`                  | 存放 Dialect 的实现（`StandaloneDialect.cpp`）       |
| `standalone-opt/`       | 一个可执行工具，类似你想写的 `my-opt.cpp`            |
| `standalone-plugin/`    | 演示如何用 plugin 加载 Dialect（高级用法，可先忽略） |
| `standalone-translate/` | 演示 `mlir-translate` 风格的工具                     |
| `test/`                 | 存放 `.mlir` 测试文件和 `FileCheck` 脚本             |

**基于这个模板开始项目：**

**复制整个 `llvm-project/mlir/examples/standalone` 文件夹为你自己的项目：**

```
cp -r llvm-project/mlir/examples/standalone ~/my-mlir-project
```

(1)重命名其中内容，比如：

- `StandaloneDialect` → `MyDialect`
- `standalone-opt` → `my-opt`

(2)在 `CMakeLists.txt` 和 `td/cpp/h` 文件中全局替换命名空间 / 类名 / 路径。