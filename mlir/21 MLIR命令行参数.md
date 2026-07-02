```c++'
::mlir::Pass::Option<uint64_t> level{
    *this, 
    "level", 
    ::llvm::cl::desc("log level")
};
```

#### 1 、`::mlir::Pass::Option<uint64_t>`？

 **模板类**

```bash
my-opt --level=2
```

```c++
Option<uint64_t> level{*this, "level", llvm::cl::desc("log level")};
```

#### 2、参数的含义

| 参数                          | 类型            | 含义                        |
| ----------------------------- | --------------- | --------------------------- |
| `*this`                       | `PassWrapper &` | 告诉这个选项它属于哪个 Pass |
| `"level"`                     | `const char*`   | `--level`                   |
| `llvm::cl::desc("log level")` | 描述对象        | 说明文字                    |

#### 3、怎么用  

- `Option<int>`类型的成员变量`level`
- Pass就有了选项
- this->level对其进行一定的操作

```c++
class InitPass :public  InitBase {
public:
    InitPass(){}
    void runOnOperation() override {
        module::init_loglevel(this->level);
    }
}
```

