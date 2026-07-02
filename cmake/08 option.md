# 1. **option**

`option` 是 CMake 用来定义**布尔类型开关参数**的指令。

类似于 C++ 里的：

```
bool enable_test = false;
```

它允许用户在运行 CMake 时通过命令行控制某些功能是否开启。

------

#  2. option 的语法

```
option(<变量名> "帮助信息" <默认值>)
```

- `<变量名>`：CMake 中使用的变量
- `"帮助信息"`：在 cmake-gui 或 ccmake 中显示
- `<默认值>`：`ON` 或 `OFF`

