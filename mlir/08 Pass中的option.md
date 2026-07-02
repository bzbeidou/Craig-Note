在tablegen里，options用来定义命令行参数

这样你就可以在命令行调用 `my-opt` 的时候写类似：

```bash
bin/my-opt --init="--freq=1000 --weight-in-mem --level=2" test.mlir

```



```
let options = [
  Option<"freq", "freq", "uint64_t", /*default=*/"",
         "chip frequance">,
  Option<"weight_in_mem", "weight_in_mem", "bool", /*default=*/"false",
         "whether to save weight in memory instead of hard disk.">,
  Option<"level", "level", "uint64_t", /*default=*/"",
         "log level">,
];
```

- 首先其中有三个Option，说明可以配置三个命令行参数

- `Option<"freq", "freq", "uint64_t", ..., "chip frequance">`
	- 第一个 `"freq"`
	- 第二个 `"freq"`：命令行上的参数名字 `--freq`，一般设置成和上面一样的就行
	- `"uint64_t"`：参数类型（支持 `bool`、`int`、`uint64_t`、`std::string` 等）
	- `/*default=*/""`：默认值，字符串形式
	- `"chip frequance"`：帮助信息（`--help` 时显示）

- `weight_in_mem` → bool 选项（默认为 false）
- `level` → 整数选项

### 生成的代码

```c++
struct InitPassOptions {
  uint64_t freq;
  bool weight_in_mem;
  uint64_t level;
};

struct InitPass : public PassWrapper<InitPass, OperationPass<ModuleOp>> {
  InitPass() {
    // 默认值初始化
    freq = 0;
    weight_in_mem = false;
    level = 0;
  }

  // options 生成的字段
  Option<uint64_t> freq{
      *this, "freq", llvm::cl::desc("chip frequance"), llvm::cl::init(0)};
  Option<bool> weight_in_mem{
      *this, "weight_in_mem",
      llvm::cl::desc("whether to save weight in memory instead of hard disk."),
      llvm::cl::init(false)};
  Option<uint64_t> level{
      *this, "level", llvm::cl::desc("log level"), llvm::cl::init(0)};

  void runOnOperation() override {
    // 可以直接用这些参数
    llvm::outs() << "freq = " << freq << "\n";
    llvm::outs() << "weight_in_mem = " << weight_in_mem << "\n";
    llvm::outs() << "level = " << level << "\n";
  }
};

```

