

```cmake
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set (CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install"
        CACHE PATH "default install path" FORCE)
endif()
```



#### 1、 `CMAKE_INSTALL_PREFIX` 是什么？

 **安装路径前缀**，默认值一般是：

- Linux：`/usr`
- Windows：`C:/Program Files`
- Mac：`/usr/local`

**这里的安装路径可以理解为对build构建的产物进行了一定的提炼，本来构建之后在build文件夹里有很多文件，但是给别人用的时候不需要这么多文件，仅仅需要一部分，安装就是这样的，多一个install文件夹里面存提炼的内容**

- **使用cmake命令的时候可以直接用这个选项进行设置**

```
cmake -DCMAKE_INSTALL_PREFIX=/my/path ..
```

#### 2、 `set(... CACHE PATH ... FORCE)` 是什么

```
set (CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install"
        CACHE PATH "default install path" FORCE)
```

把安装路径设为：

```
${CMAKE_BINARY_DIR}/install
```

例如你的构建目录是：

```
build/
```

那么 CMAKE_INSTALL_PREFIX 就会变成：

```
build/install
```

安装时所有生成文件（库文件、可执行文件）都会被安装到该目录。

3.2 `CACHE` 的意思

把变量写入 CMake 的 cache（`CMakeCache.txt`），这样下次 CMake 运行时仍然有效。

3.3 `PATH` 的意思

告诉 CMake 这个变量的类型是 **路径类型**，用于 GUI 和参数提示。

3.4 `FORCE` 的意思

无论之前cache里是否已经有值，都强制覆盖。