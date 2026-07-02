```c++
  auto from = slice0_op.getInput();
if (from != slice1_op.getInput() ||
      std::distance(from.user_begin(), from.user_end()) != 2) {
    return failure();
  }
```

```
%a = top.Conv ...
%b = top.Add %a, %x
%c = top.Mul %a, %y
```

-  (`top.Add` 算子)
-  (`top.Mul` 算子)

有如下 MLIR 片段：

```
%a = "top.Input"() : () -> tensor<4xf32>
%b = "top.Add"(%a, %a) : (tensor<4xf32>, tensor<4xf32>) -> tensor<4xf32>
%c = "top.Mul"(%a, %a) : (tensor<4xf32>, tensor<4xf32>) -> tensor<4xf32>
```

```
Value a = ...; // %a
for (auto user : a.getUsers()) {
  llvm::errs() << user->getName() << "\n";
}
```

```
"top.Add"
"top.Mul"
```

```
std::distance(a.user_begin(), a.user_end()) == 2
```
