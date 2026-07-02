在mlir基础知识里学习过block

```
ModuleOp
  └── FuncOp
        ├── Block_0
        │     ├── Operation_0
        │     ├── Operation_1
        │     └── ReturnOp
        └── Block_1 (可选)
```

- 一个 `FuncOp`（函数）里可以有多个 Block；
- Block 包含若干 Operation；
- Operation

#### 1 实际代码

```c++
Block &entryBlock = fn.front();
```

**FuncOp的内部就是由block组成**

（通常就是入口块）。

`entryBlock.back()` 

`entryBlock.back()` 

```mlir
func.func @main() {
  %0 = "tpu.add"(%a, %b)
  %1 = "tpu.relu"(%0)
  return %1
}
```

```
FuncOp
 └── Block
      ├── Operation("tpu.add")
      ├── Operation("tpu.relu")
      └── Operation("return")

```

```;c++
Block &b = fn.front();         
Operation &op = b.back();       
```

