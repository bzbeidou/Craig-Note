```cmake
find_package(libsophon REQUIRED)
include_directories(${LIBSOPHON_INCLUDE_DIRS})

add_executable(${YOUR_TARGET_NAME} ${YOUR_SOURCE_FILES})

target_link_libraries(${YOUR_TARGET_NAME} ${the_libbmlib.so})
```



## 1 `find_package()`

`find_package(libsophon REQUIRED)` 

- 头文件路径（include dirs）
- 库文件路径（动态库或静态库 .so / .a）
- 编译选项
- 依赖的其他库
- 版本信息
- CMake 导出的目标（例如 libsophon::bmrt）

把它理解成“找一个复杂模块，而不仅是一个库文件”更准确。

这句代码具体做了什么？

```
find_package(lib REQUIRED)
```

表示：

1. 让 CMake 在系统里搜索名为 **lib 的包
2. 如果找不到，就报错（因为用了 `REQUIRED`）
3. 找到了后，会自动加载这个包里定义的：
	- include 路径
	- library 路径
	- CMake target
	- 其他配置

## 2 TPU-MLIR

```
find_package(MLIR REQUIRED CONFIG)
find_package(LLVM REQUIRED CONFIG)
```

里面定义了大量 target，比如：

```
MLIRFuncDialect
MLIRIR
LLVMCore
LLVMSupport
```

才能写：

```
target_link_libraries(tpuc-opt PRIVATE
    MLIRIR
    MLIRFuncDialect
    LLVMCore
    LLVMBitReader
)
```

## 3、CMAKE_MODULE_PATH `变量

 **列表目录**

```
list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
```

还要去 /yourproject/cmake 搜

```
list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
find_package(libsophon REQUIRED)
```
