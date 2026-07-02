# 1、llvm项目未构建与构建之后的区别

| 子项目    | 说明                                    |
| --------- | --------------------------------------- |
| `llvm/`   | LLVM 核心（优化器、IR、目标架构后端等） |
| `mlir/`   | MLIR 多层 IR 编译框架                   |
| `clang/`  | Clang C/C++ 编译器                      |
| `lld/`    | 链接器                                  |
| `flang/`  | Fortran 编译器                          |
| `libcxx/` | C++ 标准库实现                          |

也就是直接下载得到的并不是可执行文件，还没有编译成可执行程序

**1.2 使用Ninja/Make构建之后，才会生成可执行命令行工具**

可执行命令行工具（如 `clang`, `mlir-opt`, `llc` 等）

简单构建举例

```
cmake ../llvm \
  -G Ninja \
  -DLLVM_ENABLE_PROJECTS="mlir;clang" \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=../install
ninja install
```



构建完成大致得到如下内容：

```
install/
├── bin/
│   ├── clang             ← C/C++ 编译器
│   ├── mlir-opt          ← MLIR 优化器（运行 Pass）
│   ├── mlir-translate    ← MLIR 转换器
│   ├── llc               ← 生成汇编代码
│   ├── opt               ← LLVM 优化器
├── lib/
│   ├── libMLIR*.so       ← MLIR 的共享库
│   ├── libLLVM*.so       ← LLVM 的共享库
├── include/
│   ├── llvm/             ← LLVM 头文件
│   ├── mlir/             ← MLIR 头文件
├── python_packages/
│   └── mlir/             ← Python 模块（可 `import mlir`）
```

# 2、c++变成python调用的库

C++ 代码无法被 Python 直接识别，需要通过一套叫做 **“语言绑定（Language Binding）”** 的机制，把 C++ 函数 **“包一层”**，变成 Python 能识别的形式。最常用Pybind11编写绑定代码

```
[C++ 实现逻辑]
        ↓
[使用 Pybind11 编写绑定代码]
        ↓
[CMake 构建生成 _xxx.so 动态链接库]
        ↓
[Python 通过 import xxx 加载这个 .so 文件]
        ↓
[调用 C++ 实现的函数或类]
```

```c++
// mycalc.cpp
int add(int a, int b) {
    return a + b;
}
```

但这段代码 **Python 完全不能识别**。我们要用 **Pybind11 包装**它。

```c++
// bindings.cpp
#include <pybind11/pybind11.h>
namespace py = pybind11;

// C++ 原函数
int add(int a, int b) {
    return a + b;
}

// 绑定函数
PYBIND11_MODULE(mycalc, m) {
    m.def("add", &add, "A function which adds two numbers");
}
```

- 用 `PYBIND11_MODULE(mycalc, m)` 声明一个 Python 模块名 `mycalc`
- 把 C++ 的 `add` 函数暴露给 Python
- Python 中以后可以这样使用：`mycalc.add(3, 4)`

```
# CMakeLists.txt
cmake_minimum_required(VERSION 3.14)
project(mycalc)

add_subdirectory(pybind11)  # 假设你把 pybind11 clone 到这个目录
pybind11_add_module(mycalc bindings.cpp)
```

构建之后会生成一个：

```
mycalc.cpython-310-x86_64-linux-gnu.so
```

这是 Python 能加载的模块！

```c++
# test.py
import mycalc

print(mycalc.add(10, 20))  # 输出 30
```

Python 成功调用了 C++ 实现的 `add` 函数。

# 3、tpu-mlir项目的使用

**（1）`envsetup.sh`**

在容器中使用这个脚本：

```
source ./envsetup.sh
```

```
$ echo $PROJECT_ROOT
/workspace/tpu-mlir

$ which tpuc-opt
/workspace/tpu-mlir/install/bin/tpuc-opt

```

**（2）build.sh**

主构建脚本**，把源码编译成可执行程序和动态库

`build.sh`  `cmake` 和 `cmake --build`

先看 `build.sh` 中的两个关键部分：

**配置阶段（相当于 `cmake ..`）**

```
cmake -G Ninja \
  -B "${BUILD_PATH}" \
  -DCMAKE_C_COMPILER=clang \
  -DCMAKE_CXX_COMPILER=clang++ \
  -DCMAKE_BUILD_TYPE="${BUILD_TYPE}" \
  -DCMAKE_CXX_FLAGS="${CXX_FLAGS}" \
  ...
  "${PROJECT_ROOT}"
```

这是自己手动写 `cmake ..` 时常见的配置，只不过：

- `-B "${BUILD_PATH}"`：指定构建目录
- `-G Ninja`：使用 Ninja 构建系统而不是默认的 Make
- `-DXXX=YYY`：设置各种变量，比如 `CMAKE_BUILD_TYPE` 设置为 `Debug`

**构建安装阶段（相当于 `cmake --build`）**

```
cmake --build $BUILD_PATH --target install -j${cpu_num}
```

这等价于你平常写的：

```
cd build
cmake --build . --target install -j4
```



### 3.1关于执行build.sh的问题   

软链接，也叫符号链接（symlink），是 Linux/Unix 系统中的一种特殊文件：

- 它本身并不存储实际文件内容，而是保存了另一个文件的路径。
- 当你访问软链接文件时，系统会自动跳转到它指向的目标文件，就像快捷方式一样。

```
ln -s libbmlib.so.0 libbmlib.so
```

执行这个命令就把libbmlib.so这个文件 链接到了libbmlib.so.0这个文件（注意libbmlib.so 一定是不能提前存在的，而是通过这条命令创建了文件并且链接到）

### 3.2 llvm-project的下载构建过程

（1）先git克隆

（2）然后进入到llvm-project

（3）创建一个build文件夹然后进入

（4）执行cmake 从而得到一些配置文件

（5）再执行make或者ninja install完成构建

**注意：可能会构建爆掉内存**

默认情况下，`cmake --build . --target install` 会自动检测 CPU 数量，**最大并发构建线程数可能远超你的内存承受范围**，所以建议手动控制。

比如你有 16GB 内存，建议最多使用 2～4 个线程：

```
cmake --build . --target install -j2
```

`-j2` 表示只启动 2 个并发线程构建。

```
cmake --build . --target install
```

这条命令可以分为三个部分：

| 部分               | 含义                                                         |
| ------------------ | ------------------------------------------------------------ |
| `cmake --build .`  | 表示 **构建当前目录下的工程（这里的 `.` 是当前目录，也就是 `build` 目录）** |
| `--target install` | 指定构建的目标是 `install`，也就是执行 `CMakeLists.txt` 中定义的 `install()` 操作 |
| 组合               | 意思是“**构建并执行 install 操作**”，通常就是把编译好的文件安装到你设置的安装路径（例如 `../install`） |

在传统 C++ 项目中，使用 `Makefile` 构建时你会写：

```
make
make install
```

而使用 `CMake` 的现代项目，推荐统一用 `cmake --build` 来代替 `make`、`ninja` 等构建系统的调用：

 所以可以认为：cmake --build . --target install ≈make install 或 ninja install

而 `cmake --build .` ≈ `make` 或 `ninja`

**为什么要这样写**

因为CMake是一个跨平台的构建工具生成器，支持多种构建后端：`make`、`ninja`等等

所以只要用`cmake --build .` 那么就会自动调用我们在`cmake`配置时选择的构建工具

**`--target install`是什么意思？**

这个 `target` 是 `CMakeLists.txt` 中定义的目标。例如：

```
install(TARGETS my_tool DESTINATION bin)
```

执行 `--target install` 就会把构建出来的 `my_tool` 安装到指定目录。

**举例：**

```
$ mkdir build && cd build
$ cmake -G Ninja .. -DCMAKE_INSTALL_PREFIX=../install
$ cmake --build . --target install
```

执行之后会看到：

```
build/
└── ...
install/
    ├── bin/
    │   └── mlir-opt
    ├── lib/
    │   └── libMLIR.so

```

这些就是安装好的可执行程序和库
