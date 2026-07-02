pip就是安装python库的，但是这只是一个广义的定义，其中可以装python代码或者一些可执行命令文件



#### 1、最常见的pip  比如pip numpy

```bash
pip install numpy
```

这些库确实主要是 `.py` 源代码文件。



### 2、像`pip install tpu-mlir`这种就不仅仅是一个python库，而是一个工具链，包含如下

```
tpu-mlir/
├── model_transform.py    # Python 脚本
├── model_deploy.py       # Python 脚本  
├── calibration.py        # Python 脚本
├── tpu_compiler          # 编译好的二进制程序（C++）
├── libtpu_mlir.so        # 编译好的库文件（C++）
└── tpu_mlir/            # Python 包目录
    ├── __init__.py
    └── python_api.py
```





类比理解如下：

| 项目     | 纯 Python 库          | tpu-mlir                  |
| :------- | :-------------------- | :------------------------ |
| **内容** | 只有 .py 文件         | .py 脚本 + 二进制程序     |
| **安装** | `pip install library` | `pip install tpu_mlir`    |
| **使用** | `import library`      | 运行 `model_transform.py` |
| **性质** | 库                    | 工具链                    |