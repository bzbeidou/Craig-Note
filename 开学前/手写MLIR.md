## mlir

#### 1、手写

```
func.func @add(%a: f32, %b: f32) -> f32 {
  %sum = arith.addf %a, %b : f32
  return %sum : f32
}
```

 解释

- `func.func` 是定义一个函数的 Operation，来自 `func` Dialect
- `%a` 和 `%b` 是函数参数（像函数的输入变量）
- `arith.addf` 是浮点加法（来自 `arith` Dialect）
- `%sum` 是中间结果变量（使用了 SSA 语法）
- `return` 返回结果

```
func.func @add(%a, %b) -> f32   ← 顶层 Operation（函数）
└── Block (入口块)
    ├── %sum = arith.addf %a, %b : f32    ← 子 Operation
    └── return %sum : f32                ← 子 Operation

```

**func.func**是一个 Operation（Op）

-  `func.func ... { ... }` 是一个 **函数定义**
- `func` 是 Dialect 的名字
- `func`的名字叫 `"func"`，放在 `func` Dialect 里

**`@add` 是这个函数的名字**，MLIR 用 `@名字` 表示一个全局的标识符。

- `@add` 是函数名
- 在调用函数时也会写成 `call @add(...)`

**`{ ... }` 是什么？**

 是这个函数的“代码体

- MLIR 中，每个 Op 可以包含 **Region**
- `Region` 就像一个“代码块容器”，里面装着代码
- Region 里面可以有多个 **Block**（代码块）
- 每个 Block 是一堆连续执行的 Op

这个函数 `func.func` 有：

-  Region
- Region 里有 1 个 Block
- Block 里有 2 个 Op

```mlir
func.func @add(%a: f32, %b: f32) -> f32 {   // 这是一个 Operation，名字叫func.func 
  ^bb0(%a: f32, %b: f32):                   // 这是一个 Block，名字默认叫 ^bb0
    %sum = arith.addf %a, %b : f32         // 这是一个 Operation
    return %sum : f32                      // 又一个 Operation
}
```

```mlir
func.func @add(%a: f32, %b: f32) -> f32 {
  %sum = arith.addf %a, %b : f32
  return %sum : f32
}
```

```
"func.func"() ({
  ^bb0(%a: f32, %b: f32):
    %sum = "arith.addf"(%a, %b) : (f32, f32) -> f32
    "func.return"(%sum) : (f32) -> ()
}) {sys_name = "add", func_type = (f32, f32) -> f32}: () -> ()

```

特点：

- 所有 Operation 用 `"Op名"` 明确写出（用字符串表示）
- 所有类型都在 `:` 后明确写出“输入类型 -> 返回类型”
- Block 要显式写出（`^bb0(...)`）



#### 2、mlir中一切都是Op含义

不管定义还是运算都是Op，只是可以分为不同的Op

| 代码         | 是什么？  | 是什么类型的 Op？ |
| ------------ | --------- | ----------------- |
| `func.func`  | 是一个 Op | 用于“定义函数”    |
| `arith.addf` | 是一个 Op | 执行浮点加法      |
| `return`     | 是一个 Op | 函数返回语句      |



#### 3、定义自己的Dialect的时候不需要自己定义函数这种dialect

MLIR 已经有了现成的标准 Dialect，比如 `func` Dialect

`func.func` 已经可以很完美地表示“函数定义”

自己定义的 Dialect 主要是为了表示**自定义的语义**（比如特定算子、Tensor操作、图优化Op等）



## 三、mlir语法部分



#### 引、mlir有没有严格的语法要求？为什么看到的这里的函数定义的Op和addf的Op写法不一样？

```
func.func @add(%a: f32, %b: f32) -> f32 {
  %sum = arith.addf %a, %b : f32
  return %sum : f32
}
```

**有标准写法，但是用了语法糖，看起来不统一**



发现了：

- `arith.addf` 这样的 **Op 写法** 是：
	 `%result = dialect.opname operands : type`
- 但像 `func.func @foo(...) -> ... {}` 这种，**看起来不像是上面那种格式**

疑问：

> ❓ 是不是所有 Op 都得写成 `%x = ... : ...` 这样的格式？
>  ❓ MLIR 到底有没有“语法标准”？为什么写法风格不统一？

**答案是：**

>  MLIR 有语法标准，而且非常严格！
>  所有 Op 的写法都有 **统一结构**，你看到的不统一，其实是**语法糖（简化写法）**在起作用。



#### 1、MLIR 的核心语法结构（统一写法）

MLIR 中，所有 Operation 的标准结构如下：

```
%result = "dialect.opname"(%operand1, %operand2) : (input_types) -> result_type
```

这叫做 **"通用语法"（Generic Syntax）**，是 MLIR 最原始、最通用、最严格的语法。所有 Op 都可以用这种写法。

```
%sum = "arith.addf"(%a, %b) : (f32, f32) -> f32
```

这表示：

- 一个名字是 `"arith.addf"` 的 Op
- 它的两个输入是 `%a` 和 `%b`，类型是 `f32`
- 它的输出结果是一个 `f32`
- 把结果绑定到 `%sum`

**函数定义也可以写成这种形式：**这样看起来就统一了不是吗

```
"func.func"() ({
  ^bb0(%a: f32, %b: f32):
    %sum = "arith.addf"(%a, %b) : (f32, f32) -> f32
    "func.return"(%sum) : (f32) -> ()
}) {sym_name = "add"} : () -> (f32)

```



**因此，完整的MLIR Op结构如下**

```
%result = "dialect.op_name"(<operands>) {<attributes>} : (<input types>) -> (<result types>)
              ^             ^         ^             ^                   ^
           名字字符串     操作数    可选属性     输入类型列表      结果类型列表

```

这条规则适用于所有 Op！包括你看到的 `arith.addf`、`func.func`，甚至是 `func.return`。

但一些复杂的 Op（比如定义函数的 `func.func`）还带有 **Region**，就是那个 `({...})`。



#### 2、详解差异

① `func.func` 后面为啥有 `({ ... })`？这是 **Region**

```
"func.func"() ({
  ^bb0(%a: f32, %b: f32):
    ...
})
```

>  这是因为 `func.func` 这个 Op 定义中**包含一个 Region（区域）**，它表示函数体！

- 这个 Region 就像一个函数体的大括号。
- Region 中有一个或多个 **Block**（基本块）
- `^bb0(...)` 就是 block 的名字（`bb0` 是基本块 0）。

所以，`({...})` 并不是语法上的例外，而是**这类 Op（带函数体）的一种标准结构**。

② `"arith.addf"` 为什么不带 `{}`？

```
%sum = "arith.addf"(%a, %b) : (f32, f32) -> f32
```

因为 `arith.addf` 是一个**普通算子，没有 Region**！

它的行为是单纯计算，不包含代码块，所以它就用标准格式书写即可：

```
%res = "dialect.op_name"(operands) : (input_types) -> result_types
```

- 没有 `{}` 是因为它 **没有 region**
- 没有 `sym_name` 是因为它 **不是一个命名实体（不是函数、模块等）**

③ `{sym_name = "add"}` 是啥？—— 这是一个 **属性（Attribute）**

```
"func.func"() (...) {sym_name = "add"} : () -> ()
```

这是说：

- 给 `func.func` 这个 Op 添加一个属性：`sym_name`，值为 `"add"`
- 它告诉编译器：这个函数叫 `@add`

 这其实是对 `func.func @add(...)` 的语法糖展开！

⑤ 为啥 `%sum = arith.addf ...` 和 `func.func(...)` 差别这么大？

因为：

| Op 类型       | 用途                               | 写法区别                           |
| ------------- | ---------------------------------- | ---------------------------------- |
| `arith.addf`  | 纯粹的计算操作                     | 没有 Region，不需要名字            |
| `func.func`   | 带 Region 的结构性操作（定义函数） | 需要包含 Block、Region、名字等信息 |
| `func.return` | 终结块的操作                       | 不能再接下一条指令，没有结果值     |



#### 3、这里是定义函数Op，并不是调用函数Op

很简单，因为这是定义函数，没有调用，调用时才会有接收，block里的内容当然是有接收的

```
"func.func"() ({
  ^bb0(%a: f32, %b: f32):
    %sum = "arith.addf"(%a, %b) : (f32, f32) -> f32
    "func.return"(%sum) : (f32) -> ()
}) {sym_name = "add"} : () -> ()

```



#### 4、为什么所有类型都要写出来

**因为mlir中的Op是强类型机制**，必须明确说明接受什么类型，返回什么类型

#### 5、Op的Type和值的Type

OP居然也有Type！

但不是所有 Op 都有类型，只有那些 **“值返回型”的 Op** 才有「值」和「类型」的概念。

MLIR 中有两种“Type”：

1. **值的 Type（Value Type）**：这是最常见的，比如 `%x: f32` 里的 `f32`
2. **Op 本体的 Type（Op Type）**：指的是整个 Op 代表的“操作”的输入输出类型（通常写在 `: (...) -> (...)` 部分）

**（1）值返回型的Op  **

```
%sum = "arith.addf"(%a, %b) : (f32, f32) -> f32

```

- 这里 `arith.addf` 是一个 Op。

- 它输入两个 f32，输出一个 f32。

- 它的 **Op 类型是 `(f32, f32) -> f32`**，表示输入输出签名。

- 它产生一个值 `%sum`，这个值的类型是 `f32`（也就是 Value Type）。

**（2）定义型的Op（比如 func.func）**

```
"func.func"() (...) {function_type = (f32, f32) -> f32} : () -> ()

```

- 它“输出”一个函数值 → 返回类型是 `(f32)`，即函数返回一个 `f32`。

- 所以 Op 的 Type 是：`() -> ()`。



**（3）**

```
"func.return"(%x) : (f32) -> ()
```

- 它有一个输入 f32，它的 Type 是 `(f32) -> ()`。



#### 5、Op的Attributes  关键就是用{}里的键值对来表示

`callee`、`function_type` 等都是 **属性（Attributes）**，它们 **不是所有 Op 通用的字段，而是每个 Op 自己定义的字段**。



**（1）什么是MLIR中Op的Attribute**

在 MLIR 中，每个 Operation 都可以带一些额外信息，比如：

- 名字（symbol）
- 类型信息（function_type）
- 被调用的函数名（callee）
- 是否是常量（value = 42）

这些就是 **Attributes**，语法是写在花括号 `{}` 里的键值对：

```
"func.call"(%x, %y) {callee = @add} : (f32, f32) -> f32
```

其中的 `{callee = @add}` 就是 attribute。

**（2）在tablegen文件中是什么形式**

** `func.call` Op：**

它的 attribute 定义类似于：

```
let arguments = (ins ...);
let attributes = (ins SymbolRefAttr:$callee);
```

就是说：这个 Op 需要一个属性 `callee`

**（3） `func.func` Op：**

```
let attributes = (ins TypeAttr:$function_type, StrAttr:$sym_name);
```

这个函数定义 Op 会包含这些属性：

- `sym_name` → 这个函数的名字
- `function_type` → 它的函数类型 `(f32, f32) -> f32`



## 四、继续手写  熟悉mlir并使用多个dialect

#### 1、练习1  手写 MLIR 定义加法函数并调用

定义一个函数 `add` 接受两个 f32 参数并返回它们的和。然后在 `main` 中调用它，获取返回值。

```mlir
// 定义函数 add
"func.func"() ({
  ^bb0(%a: f32, %b: f32):
    %sum = "arith.addf"(%a, %b) : (f32, f32) -> f32
    "func.return"(%sum) : (f32) -> ()
}) {sym_name = "add", function_type = (f32, f32) -> f32} : () -> ()

// 定义 main 函数，调用 add
"func.func"() ({
  ^bb0:
    %x = arith.constant 3.0 : f32
    %y = arith.constant 4.0 : f32
    %z = "func.call"(%x, %y) {callee = @add} : (f32, f32) -> f32
    "func.return"(%z) : (f32) -> ()
}) {sym_name = "main", function_type = () -> f32} : () -> ()

```



**问题：MLIR 中 `function_type = (f32, f32) -> f32` 和 `: () -> ()`两个看起来都像“函数类型”，看起来有什么区别？**

| 部分                                | 类比 C 的哪一块               | 含义                                                         |
| ----------------------------------- | ----------------------------- | ------------------------------------------------------------ |
| `function_type = (f32, f32) -> f32` | `float add(float a, float b)` | 表示这个函数接受两个 f32，返回一个 f32                       |
| `: () -> ()`                        | 无对应                        | 表示这个 `func.func` 这个“定义操作”本身没有输入也没有返回值（不是函数调用！） |

MLIR 中每个 Operation 都是一个表达式，它的 “类型” 结构总是 `: (输入值类型) -> (返回值类型)`，哪怕什么都没有，也要写成 `() -> ()`。这里是定义这个函数，当然没有具体的输入和输出，所以都是()。

但是是通过`function_type = (f32, f32) -> f32`来规定函数的输入和输出的，`: () -> ()`的部分是表示当前这个Op的具体的输入和输出的，是一种规定，不管是定义型Op还是操作型Op都得这样写

| Op 类型       | 举例           | Op 本身输入/输出                   | 会不会产生值 | 类型写法                             |
| ------------- | -------------- | ---------------------------------- | ------------ | ------------------------------------ |
| **定义型 Op** | `"func.func"`  | 没有输入输出（本质是“建一个结构”） | 不产生值     | `: () -> ()` 或 `: () -> (返回类型)` |
| **操作型 Op** | `"arith.addf"` | 输入是 2 个 f32，输出是 1 个 f32   | 产生值       | `: (f32, f32) -> f32`                |
| **调用型 Op** | `"func.call"`  | 输入是实参，输出是返回值           |              | `: (T1, T2, ...) -> (R1, R2, ...)`   |

**问题：`{callee = @add}` 是什么？**

这是一个 **属性（attribute）列表**，不是 region。

#### 2、练习2 判断一个浮点数是否为正数

我们写一个 MLIR 函数，它接收一个 `f32` 类型的数 `x`，
 如果 `x > 0.0`，返回 `1.0`，否则返回 `0.0`。

```mlir
"func.func"()({
    ^bb0(%a : f32):
        %zero = "arith.constant"() {value = 0.0 : f32} : () -> f32
        %cond = "arith.cmpf"(%a, %zero){predicate = #arith.cmpf<ogt>} : (f32, f32) -> i1
        %one = "arith.constant"() {value = 1.0 : f32} : () -> (f32)
        %zero2 = "arith.constant"() {value = 0.0 : f32} : () -> (f32)
        %result ="arith.selecet"(%cond, %one, %zero2) : (i1, f32, f32) -> (f32)
        "func.return"(%result) : (f32) -> ()
}){sys_name = "is_positive", function_type = (f32) -> (f32)} : () -> (f32) 
```

#### 3、练习3 使用loop实现加法累加

要求：手写一个MLIR文件，实现从0累加到N的加法

涉及到的Dialect

| Dialect | Op               | 说明            |
| ------- | ---------------- | --------------- |
| `func`  | `func.func`      | 定义函数        |
| `func`  | `func.return`    | 返回值          |
| `arith` | `arith.constant` | 常量            |
| `arith` | `arith.addi`     | 整数加法        |
| `scf`   | `scf.for`        | 结构化 for 循环 |
| `scf`   | `scf.yield`      | 循环体返回值    |

```mlir
module {
  "func.func"() ({
    ^bb0(%n: i32):
      %zero = "arith.constant"() {value = 0 : i32} : () -> i32
      %one = "arith.constant"() {value = 1 : i32} : () -> i32

      // scf.for %i = %zero to %n step %one
      %result = "scf.for"(%zero, %n, %one) ({
        ^bb1(%i: i32, %sum: i32):
          %new_sum = "arith.addi"(%sum, %i) : (i32, i32) -> i32
          "scf.yield"(%new_sum) : (i32) -> ()
      }) : (i32, i32, i32) -> i32

      "func.return"(%result) : (i32) -> ()
  }) {
    sym_name = "sum_up_to_n",
    function_type = (i32) -> i32
  } : () -> ()
}

```
