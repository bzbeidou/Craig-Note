#### 1、linux环境变量控制 

```
export LLVM_DIR=~/mlir_toy/llvm-project/build/lib/cmake/llvm
export MLIR_DIR=~/mlir_toy/llvm-project/build/lib/cmake/mlir

```

#### 2、CMakeLists

**（1）project()   声明工程用的**

`project(mlir-toy ...)`：声明这是一个名为 `mlir-toy` 的 **CMake 项目**。

在 CMake 中，每一个构建工程（无论多复杂）都需要一个 `project(...)` 来定义项目的名称。

这里还可以设置版本、语言等，有选项可以选

```
project(mlir-toy VERSION 0.0.0 LANGUAGES CXX)
这里设置了mlir-toy名称的一个项目工程，给他的版本是0.0.0，然后语言是c++

写了这些额外的选项之后，CMake会自动定义一些变量可以用，前缀就是这里的工程名
比如这里写了VERSION，那么CMake 自动定义以下变量（你可以在后面用）：
mlir-toy_VERSION：等于 0.0.0
mlir-toy_VERSION_MAJOR：等于 0
mlir-toy_VERSION_MINOR：等于 0
mlir-toy_VERSION_PATCH：等于 0
```

**（2）set  就是用来定义变量的，也可以指定**

```
set(MY_NAME "keke")
message(STATUS "Hello, ${MY_NAME}")

这里就是把"keke"这个字符串赋值给MY_NAME，
输出：
-- Hello, keke

```

*常用*

*（1）设置编译器选项（如 C++ 版本）*

```
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED YES)
```

*（2） 定义自定义路径变量*

```
set(MLIR_DIR "/home/user/llvm/build/lib/cmake/mlir")
```

*（3）定义布尔开关*

```
set(BUILD_EXAMPLES ON)
```

**（3）find_package 查找加载外部的包**

比如需要用到MLIR库里的一些头文件等，就需要用到find_package来找到MLIR的位置

```
find_package(<PackageName> [version] [REQUIRED] [QUIET] [MODULE|CONFIG])


<PackageName>：包名，比如这里是 MLIR

[version]：可选，要求版本号

[REQUIRED]：如果找不到包就报错，强制必须找到

[QUIET]：找不到时不输出警告

[MODULE|CONFIG]：查找方式
```

*常用的*

```
find_package(MLIR REQUIRED CONFIG)

```

*去哪找？*

```
cmake .. -DMLIR_DIR=/home/user/llvm-project/build/lib/cmake/mlir

```

**（4）list   操作列表变量的命令**

```
list(APPEND CMAKE_MODULE_PATH "${MLIR_CMAKE_DIR}")
意义：把 MLIR_CMAKE_DIR 这个目录路径，追加到 CMAKE_MODULE_PATH 变量里。

```



**（5）include   载入另一个CMake文件   类似于#include**

```
include(TableGen)
include(AddLLVM)
include(AddMLIR)

比如第一句告诉 CMake 去找名字叫 TableGen.cmake 的模块文件，然后执行其中的命令。
这个文件通常定义了一些和 LLVM TableGen 工具相关的函数和宏，方便在 CMake 中使用 TableGen 来生成代码。

```

*这些.cmake文件在哪 ？   在构建目录里*  

```
/path/to/llvm-project/build/lib/cmake/llvm/TableGen.cmake
/path/to/llvm-project/build/lib/cmake/llvm/AddLLVM.cmake
/path/to/llvm-project/build/lib/cmake/mlir/AddMLIR.cmake
```

*问:   .cmake文件是什么？*

就是用cmake脚本写的文件，和CMakeLists.txt类似，`CMakeLists.txt` 是 CMake 构建流程的入口文件，**必须有**，告诉 CMake 怎么构建项目。

`.cmake` 文件是给 `CMakeLists.txt` 使用的工具脚本文件，里面可以写函数、模块，便于代码复用和分割。一般不需要自己写

**（6）include_directories()**

给编译过程添加头文件搜索路径，等价于编译时加 `-I/path/to/include` 参数。

```
include_directories(${LLVM_INCLUDE_DIRS} ${MLIR_INCLUDE_DIRS})

```

**（7）add_executable 生成可执行文件**

```
add_executable(mlir-toy main.cpp)
```

CMake 就会自动做以下几件事：

1. **告诉编译器**：我要编译 `main.cpp`。
2. **生成目标程序**：用 `main.cpp` 生成名为 `mlir-toy` 的可执行文件。

**（8）多模块化组织项目**

*1）只用一个CMakeLists.txt文件*  学习可以先用这个

```
add_executable(toyc
  toyc.cpp
  parser/AST.cpp
  parser/Parser.cpp
  mlir/MLIRGen.cpp
  mlir/Dialect.cpp
)

```

**这样不就会获得很多可执行程序吗，并不会，只会生成main（）函数的部分，其他只是辅助**

*2）用子目录的方式  `add_subdirectory()` 组织源码*

#### 3、MLIR

**（1）MLIR Op 的结构**

**Op** 是 MLIR 中最基本的操作单元（Operation），相当于中间表示里的“指令”。

一个 Op 通常包含：

- **操作名（operation name）**，比如 `std.add` 表示整数加法。
- **操作数（operands）**，即输入的值。
- **结果（results）**，操作输出的值。
- **属性（attributes）**，不变的辅助信息，比如常量值、维度等。
- **区域（regions）**，嵌套的控制流块，可以用来表示复杂的语义。

Op 的结构定义通常在 `.td` (TableGen) 文件中描述，方便自动生成代码和验证。



**（2）MLIR图结构   -----指Operation之间的连接关系**

就是数据流和控制流构成的图结构

MLIR 通过 **函数（FuncOp）** 组织 Op，函数内部是由基本块（Block）组成，Block 中是有序的操作序列。Op 的结果可以作为其他 Op 的操作数，形成 **数据流图**，反映程序计算逻辑。

**4、从td文件生成代码的方式  两种**

*（1）直接用下载的llvm项目里的tablegen工具*

*（2）用封装好的功能，就是写在CMakeLists里的*

```
set(LLVM_TARGET_DEFINITIONS NorthStarDialect.td)

#生成头文件
mlir_tablegen(NorthStarDialect.h.inc --gen-dialect-decls -dialect=north_star)

#生成源文件
mlir_tablegen(NorthStarDialect.cpp.inc --gen-dialect-defs -dialect=north_star)

#上面是生成命令，这里是把生成的命令定义为
add_public_tablegen_target(NorthStarDialectIncGen${ch_num})
```

**补充：最后的命令是一个mlir提供的宏，也就是生成构建目标target**
