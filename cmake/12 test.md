 add_test() 后，测试成为“工程的一部分”

写：

```
add_test(NAME AddTwoNumbers COMMAND test_add)
```

你是在告诉 CMake：**这个东西是一个测试目标，不是普通程序。**

然后 CMake 会写一个结构化的测试文件：

```
build/CTestTestfile.cmake
```

里面记录：

- 测试名称
- 执行命令
- 测试是否依赖其他测试
- 工作目录
- 环境变量

**这就是结构化测试系统。**

```
test_conv
test_matmul
test_depthwise
test_lrn
test_pool
...
```

```
ctest --output-on-failure
```

CI 运行时，需要知道：

- 哪些可执行文件是测试？
- 测试失败是哪个？
- 返回码是什么？
- 输出是什么？

```
./test_add
./test_conv
./test_matmul
```

CI 完全不知道结构。

使用 CTest：

```
ctest --output-junit report.xml
```

自动生成报告交给 CI。

CTest 可以并行运行：

```
ctest -j16
```

CTest 内部知道构建目录结构：

```
ctest -C Debug
ctest -C Release
```

- Debug 编译行为正
- Release 编译也正
