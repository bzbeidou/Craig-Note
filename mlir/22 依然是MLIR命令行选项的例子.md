```c++
    module::init_loglevel(this->level);
    module::setWeightInMemFlag(weight_in_mem);
```

#### 1、整体介绍

```c++
::mlir::Pass::Option<bool> weight_in_mem{
    *this,
    "weight_in_mem",
    ::llvm::cl::desc("whether to save weight in memory instead of hard disk."),
    ::llvm::cl::init(false)
};

module::setWeightInMemFlag(weight_in_mem);
```

读取 `--weight_in_mem` 
 `module::setWeightInMemFlag()` 。

#### 2  `::mlir::Pass::Option<bool>` 前面讲过

```
my-opt --init --weight_in_mem=true
```

```cpp
::mlir::Pass::Option<bool> weight_in_mem{...};
```

名称、描述、默认值

```cpp
bool weight_in_mem = false;
```

#### 3、参数说明：

```c++
*this                 
"weight_in_mem"        
llvm::cl::desc(...)    
llvm::cl::init(false)  
```

```bash
my-opt --init test.mlir --weight_in_mem=true
```

#### 4 上面我们说的代码核心功能

```c++
module::setWeightInMemFlag(weight_in_mem);
```

把命令行参数传入模块系统，让后续逻辑知道“是否将权重保存在内存中”。

`setWeightInMemFlag` 定义如下：

```c++
void setWeightInMemFlag(bool enable) { b_weight_in_mem = enable; }
```

可见它只是设置了一个全局变量：

```cpp
bool b_weight_in_mem;
```

这样后续其他模块（例如模型导出部分）可以通过这个全局标志判断：

- 如果 `true` → 把权重存在内存中；
- 如果 `false` → 把权重写到磁盘。
