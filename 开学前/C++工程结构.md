

#### CmakeList.txt、Cmake、Makeflie、Ninja



```
你写的代码：main.cpp
        ↓
你写的描述文件：CMakeLists.txt
        ↓
CMake（生成构建脚本）
        ↓                            ↓
生成 Makefile 或 Ninja 文件    生成 Visual Studio 工程等
        ↓
make 或 ninja 执行编译命令
        ↓
可执行程序（如 ./hello）

```

**（1）Cmake**

会根据你写的 `CMakeLists.txt` 文件，**生成最终的构建配置文件**：

Linux 下生成：`Makefile`

Windows 可生成：`Visual Studio 项目`、`Ninja 构建文件`、`MSBuild` 文件等

**（2）CmakeList.txt是什么? **

`CMakeLists.txt` 有一套**规定的语法和结构规则**，就像 Python 有语法一样。它的语言叫做 **CMake 脚本语言（CMake Language）**

**了解几个常用的就行**

CMake 是 **命令驱动的语言**，每一行都是一个命令调用：

```
命令名( 参数1 参数2 ... )
比如
project(MyProject)
add_executable(main main.cpp)
所有命令不区分大小写（project 和 PROJECT 都行），但推荐用小写。
括号和参数之间不能有逗号，参数之间用空格分隔。
所有变量使用 ${变量名} 形式引用。
```

**常用命令**

| 命令                                   | 含义                |
| -------------------------------------- | ------------------- |
| `cmake_minimum_required(VERSION 3.10)` | 指定 CMake 最低版本 |
| `project(MyApp)`                       | 声明项目名称        |
| `add_executable(target sources...)`    | 添加可执行程序      |
| `add_library(libname ...)`             | 添加库文件          |
| `target_link_libraries(target lib)`    | 链接库到目标        |
| `include_directories(dir)`             | 添加头文件路径      |
| `set(VAR value)`                       | 设置变量            |

**最简单的CmakeList.txt**

```
cmake_minimum_required(VERSION 3.10)

project(HelloWorld)

add_executable(hello main.cpp)

```

**补充说明**

- CMakeLists.txt 文件必须位于项目根目录。
- 每个子目录也可以拥有自己的 CMakeLists.txt 文件。
- 多文件项目可以使用 `add_subdirectory()` 来组织。



**（3）Makeflie和Ninja   是并列的关系，Ninja编译很快**

通过Cmake处理CmakeList.txt文件,可以选择生成Makeflie或者Ninjia文件，然后这两个文件则可以通过make命令或者Ninjia命令来运行，实际上这里就调用g++进行编译



**整体流程**

写 `.cpp` / `.h` 文件（比如 `main.cpp`）

写 `CMakeLists.txt` 文件（相当于告诉 IDE/工具链该如何组织项目）

运行 CMake ➜ 自动生成 `Makefile` / `Ninja` 等

执行 `make` / `ninja` ➜ 编译 & 链接程序

**具体示例**

比如我有下面一个项目

```
项目目录：
hello_project/
├── CMakeLists.txt
└── main.cpp
可以看到里面有两个文件，一个c++文件还一个CmakeList.txt文件

文件内容
#include <iostream>

int main() {
    std::cout << "Hello, CMake + Make!" << std::endl;
    return 0;
}

CMakeLists.txt
cmake_minimum_required(VERSION 3.10)
project(HelloProject)

add_executable(hello main.cpp)

```

下面开始构建流程，根据Makefile和Ninja有两种构建方式

1）使用MakeFlie构建（这是经典做法）

```
mkdir build
cd build
cmake -G "Unix Makefiles" ..    # 生成 Makefile
make                            # 执行 Makefile，生成 hello 可执行文件
./hello                         # 运行程序

//这里在Cmake的时候就直接调用了CMakeList.txt文件
```

2）使用Ninja构建(更快)

```
mkdir build
cd build
cmake -G "Ninja" ..             # 生成 .ninja 构建文件
ninja                           # 构建
./hello

```

**两种方式都可以看到输出**

```
Hello, CMake + Make!
```





#### 典型的C++项目结构（以MLIR文件）

```
project_root/
│
├── include/              # 头文件目录（接口定义）
│   └── my_lib.hpp        # 示例头文件
│
├── src/                  # 源文件目录（实现代码）
│   └── my_lib.cpp        # 示例源文件
│
├── main.cpp              # 程序的入口函数（main函数）
│
├── CMakeLists.txt        # CMake 配置文件，管理构建流程
│
└── build/（通常不存在，构建时生成）
```

#### (1). `include/`

- **存放 `.h` 或 `.hpp` 文件（头文件）**
- 头文件定义了函数、类、结构体等的“接口”。
- 类似于 Java 中的“接口声明”或 Python 的“模块定义”。
- 比如：`my_lib.hpp` 里可能定义了一个类 `MyLib` 的声明，但不实现它。

```cpp
// include/my_lib.hpp
#pragma once
class MyLib {
public:
    void sayHello();
};
```

#### (2）. `src/`

- **存放 `.cpp` 文件（源文件）**
- 实现 `include/` 中头文件声明的函数或类。
- 比如：

```cpp
// src/my_lib.cpp
#include "my_lib.hpp"
#include <iostream>

void MyLib::sayHello() {
    std::cout << "Hello from MyLib!" << std::endl;
}
```

#### (3). `main.cpp`

- 主程序的入口文件，包含 `int main()` 函数。
- 调用 `include/` 和 `src/` 中的内容，启动程序运行。

```
// main.cpp
#include "my_lib.hpp"

int main() {
    MyLib lib;
    lib.sayHello();
    return 0;
}
```

#### (4). `CMakeLists.txt`

- 构建系统的配置脚本，告诉 CMake 如何编译这个项目。
- 会指定：
	- 哪些源文件需要编译？
	- 编译出的可执行文件名？
	- 哪些头文件路径需要包含？
	- 是否需要链接外部库？

```c++
cmake_minimum_required(VERSION 3.15)
project(MyProject)

include_directories(include)
add_subdirectory(src)

add_executable(MyApp main.cpp)
target_link_libraries(MyApp my_lib)  # 链接 src 中定义的库
```



#### 4、自定义Dialect的项目结构

```
project_root/
└── include/
    └── Dialect/
        └── NorthStar/
            └── IR/
                ├── CMakeLists.txt
                ├── NorthStarDialect.h
                └── NorthStarDialect.td

```

`IR/` 文件夹中这三个文件——`CMakeLists.txt`、`NorthStarDialect.h` 和 `NorthStarDialect.td`——正是定义 **MLIR 自定义方言（Dialect）** 的关键部分。

**(1)`NorthStarDialect.td` （最关键）**

**(2)NorthStarDialect.h**

这是 C++ 语言中**方言类的声明头文件**。

它一般包括你方言的类定义，比如：

```c++
namespace mlir {
namespace north_star {

class NorthStarDialect : public mlir::Dialect {
  ...
};

} // namespace north_star
} // namespace mlir

```

**(3)`CMakeLists.txt` （告诉 CMake 如何生成）**

这个文件告诉 CMake：

- 如何调用 `mlir-tblgen` 工具将 `.td` 转换为 `.h/.cpp`。
- 如何把这些生成文件加入到你的工程中。

**典型内容：**

```
set(LLVM_TARGET_DEFINITIONS NorthStarDialect.td)
mlir_tablegen(NorthStarDialect.h.inc -gen-dialect-decls -dialect=northstar)
mlir_tablegen(NorthStarDialect.cpp.inc -gen-dialect-defs -dialect=northstar)
add_public_tablegen_target(NorthStarDialectIncGen)

```

`mlir_tablegen(...)` 是 MLIR 提供的 CMake 函数，调用 `mlir-tblgen`。

会生成 `.h.inc` 和 `.cpp.inc`，在别处 `#include`。

`add_public_tablegen_target(...)` 是为了后续链接这些生成文件。



#### 5、TableGen    也是一个语言，可以生成c++代码

编写C++代码很麻烦，可以写TableGen比较简洁，然后用工具可以生成C++代码

**举例，TableGen文件的后缀是.td,语法与C++有点像**

```
def MyDialect : Dialect {
  let name = "mydialect";
  let cppNamespace = "::mlir::mydialect";
}

def AddOp : MyDialect_Op<"add", [NoSideEffect]> {
  let summary = "addition operation";
  let arguments = (ins I32:$lhs, I32:$rhs);
  let results = (outs I32:$result);
}
```

含义解释：

- `def MyDialect`：定义一个方言，名字是 `mydialect`，命名空间是 `mlir::mydialect`
- `def AddOp`：定义一个操作 `add`
	

然后 TableGen 就能帮你生成如下 C++：

```c++
namespace mlir::mydialect {
  class AddOp : public Op<AddOp, OpTrait::NoSideEffect> {
    Value lhs(), rhs();
    Value result();
    static StringRef getOperationName() { return "mydialect.add"; }
    ...
  };
}

```

