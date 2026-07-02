### 基本功能

1. `class` / `def` —— 定义基类、算子
2. `let` —— 给算子加属性（summary、description 等）
3. `ins` / `outs` —— 输入输出
4. 常见类型约束（`Tensor`、`I32Attr`、`StrAttr`）
5. Trait —— 声明算子特性
6. 继承 —— 复用共性（比如一类 unary/binary op）

`""` 则是普通单行字符串。

**（1）`let`**

这是最简单的：

```
let summary = "Input operator";
```

**（2）`let description = [{ ... }];`**

这里比较特殊：

```td
let description = [{
  这里写一大段注释或者解释性文字。
  可以多行。
}];
```

**（3） `let arguments = (ins ...)` 和 `let results = (outs ...)`**

这是 `TableGen` 的 DSL 语法

```td
let arguments = (ins 
  AnyRankedTensor:$input, 
  OptionalAttr<I64ArrayAttr>:$shape_tensor
);

let results = (outs AnyTensor:$output);
```

