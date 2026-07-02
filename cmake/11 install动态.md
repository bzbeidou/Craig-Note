

```cmake
add_subdirectory(cnpy)

install(FILES nntoolchain/lib/libbackend_1684.so
```



## 0 install 的作用是什么

**(1)链接**

- **可执行文件变大**
- **执行时动态链接，这样可执行文件也可以调用动态库的函数内容**

**（2）几种设置情况**

- 这样可执行文件会自动找到动态库然后找到函数
- 在某个环境变量添加这个路径
- 放置在系统的一个比较固定的目录

## **1. 什么时候执行的？**

也就是说：

```
cmake --build build
```

或者：

```
make
```

**不会触发 `install()` 指令中的复制操作。**

## 2. 只有在主动执行才会生效







