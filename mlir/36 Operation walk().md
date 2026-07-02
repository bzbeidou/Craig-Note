```c++
fn.walk([&](top::InputOp inputOp) {
});

```

#### 1、遍历

```c++
fn.walk([&](Operation *op) {

});
```

最通用的 `Operation*`

#### 2、指定类型的Operation

```c++
fn.walk([&](top::InputOp inputOp) {
});
```

```c++
template <typename OpTy, typename FuncTy>
void Operation::walk(FuncTy func);
```

当传入的lambda参数类型是一个具体的OpTy（例如 `top::InputOp`），walk内部就会自动筛选出所有满足该类型的Op

