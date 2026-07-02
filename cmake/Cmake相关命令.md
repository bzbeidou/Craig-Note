**示例：**

```cmake
cmake_minimum_required(VERSION 3.15)   # 指定 CMake 最低版本
project(MyProject VERSION 1.0)         # 项目名称和版本

set(CMAKE_CXX_STANDARD 17)             # 设置 C++ 标准
set(CMAKE_CXX_STANDARD_REQUIRED ON)    # 强制使用指定标准

add_executable(myapp main.cpp utils.cpp) # 生成可执行文件
target_include_directories(myapp PRIVATE include) # 添加头文件目录
target_link_libraries(myapp PRIVATE m)   # 链接数学库libm
```

### 1、基本项目配置

`cmake_minimum_required(VERSION 3.15)`
 要求 CMake 最低版本（防止版本太低不支持某些功能）。

`project(MyProject VERSION 1.0 LANGUAGES CXX)`
 定义项目名称、版本号和语言。

### 2、设置编译选项

`set(<var> <value>)`用来设置变量

`set(CMAKE_CXX_STANDARD 17)`
 设置 C++ 标准为 C++17。

### 3、编译

add_executable和add_library和target_include_directories和target_link_libraries
**例子：**

项目目录：

```c++
project/
  main.cpp
  math/
    math.h
    math.cpp
  CMakeLists.txt
```

**（1）如果不用add_library，那么直接写前面学的那种**

```
add_executable(MyApp main.cpp math/math.cpp)
```

```c++
add_executable(MyApp2 main.cpp math/math.cpp)
```

**（2）用add_library**

```cmake
add_library(MathLib math/math.cpp)

target_include_directories(MathLib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/math)

add_executable(MyApp main.cpp)

target_link_libraries(MyApp PRIVATE MathLib)
```

### 4、find_package  ---已经实现了的库文件包

**假设我们想用 Eigen3**

```cmake
cmake_minimum_required(VERSION 3.10)
project(UseEigenExample)

find_package(Eigen3 3.3 REQUIRED NO_MODULE)

add_executable(main main.cpp)

target_link_libraries(main PRIVATE Eigen3::Eigen)

```



**几个参数**

```cmake
find_package(<PackageName> [version] [EXACT] [QUIET] [MODULE] [REQUIRED] [COMPONENTS ...])
```

`version`指定所需库的最小版本。

后面的选项则是对于版本的容忍度：

`EXACT要求完全匹配指定版本，否则报错`

```c++
find_package(OpenCV 4.5 EXACT REQUIRED)
```

`QUIET`如果找不到，不输出错误消息（但仍然设置变量告诉你找没找到）

`REQUIRED`也是必须匹配，但是容忍度相对高，至少，如果找不到就报错并停止配置，此外代表可以配置子库，一般用不到子库这边，仅仅是用他的至少

这个REQUIRED是一般都会加上

```c++
find_package(Boost 1.70 REQUIRED COMPONENTS filesystem system)
//Boost 至少 1.70
```

```cmake
find_package(XXX REQUIRED)
```

CMake 会在自己的 **模块路径** 里去找一个叫 `FindXXX.cmake` 的文件,这个文件通常是 CMake 官方或别人写好的，用来告诉 CMake 怎么找到 `XXX` 库。



### 5、add_subdirectory

```cmake
add_library(mathLib add.cpp add.h)

target_include_directories(mathLib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
```

主工程 CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject)

add_subdirectory(math)  

add_executable(main main.cpp)

target_link_libraries(main PRIVATE mathLib)

```



### 6、list 

对cmake列表变量进行操作

### 7、include 区分 这个和添加.h依赖路径的那个 这里是包含.cmake脚本的

```c++
include(${CMAKE_SOURCE_DIR}/cmake/MyMacros.cmake)
```

这会把 `MyMacros `里的内容直接加载进来



### 8、add_library

```cmake
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            source1 source2 ... sourceN)
```

`<name>`：库的名字，比如 `MLIRTopDialect`

`source1 source2 ... sourceN`：源码文件

