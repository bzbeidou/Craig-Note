## 1、续：手写三个案例

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

      %result = "scf.for"(%zero, %n, %one) ({
        ^bb1(%i: i32, %sum: i32):
          %new_sum = "arith.addi"(%sum, %i) : (i32, i32) -> i32
          "scf.yield"(%new_sum) : (i32) -> ()
      }) {iter_args = [%zero]} : (i32, i32, i32) -> i32

      "func.return"(%result) : (i32) -> ()
  }) {
    sym_name = "sum_up_to_n",
    function_type = (i32) -> i32
  } : () -> ()
}

```



## 2、mlir语法结构

#### 1、第一层结构 Module

在 MLIR 中，**所有的代码都要放在 `module` 这个最外层容器里**。

`module` 是一个 **Operation**，它的名字是 `builtin.module`，但是在手写 MLIR 时写成 `module`。

```
module {
  // 所有的函数、变量、操作都在这里面
}
```

`module` 是 `builtin` Dialect 中定义的 Operation。全称是`"builtin.module"`

只不过 MLIR 对它做了**语法糖处理**，可以直接写成 `module {}`，以简化写法。



- 如果你手写一个 MLIR 文件，要让它能被 `mlir-opt` 或 `mlir-translate` 正确解析并执行，**必须写 `module`**。

- 如果不写 `module`，那就没有合法的顶级容器，MLIR 工具就会报错。



一个mlir文件只能有一个module（特殊情况少见），也就是module就是一个顶层容器，用来包裹函数Op以及Op





#### 2、第二层结构：Function

MLIR 中定义函数时用的是 `func` Dialect 里的 `func` 操作，即 `func.func`。

#### 3、第三层结构：操作（Op）实际上上面两层也是Op



#### 4、Block部分   控制程序的跳转  为什么要跳（看5）

**疑问是：上面案例明显两个block，那么block到底做什么用呢**

**答案是控制代码的执行顺序，比如Python代码是隐式顺序执行，但是MLIR是基于Block的运行流程，每一个复杂操作（循环、条件等）都有自己的Region，然后Region里用block表示语句**



在 MLIR 中，**block 是基本块（basic block）**：

- 每个 block 都有一个 label（比如 `^bb0`, `^bb1`）；
- 每个 block 有参数（类似函数的形参）；
- 每个 block 末尾必须以“terminator op”（例如：`scf.yield`, `func.return`, `cf.br`）结束。

*两个block的作用如下：*

| block 名 | 出现位置       | 是哪个 Region 的 block？ | 参数       | 作用                                             |
| -------- | -------------- | ------------------------ | ---------- | ------------------------------------------------ |
| `^bb0`   | 最外层         | `func.func` 的 Region    | `%n`       | 函数的唯一入口，初始化并调用 `scf.for`           |
| `^bb1`   | `scf.for` 内部 | `scf.for` 的 Region      | `%i, %sum` | for 循环体，每轮迭代执行一次，用于计算新的 `sum` |



也就是Block是MLIR里的最小跳转单位

| 特性                             | 说明                                                  |
| -------------------------------- | ----------------------------------------------------- |
| 显式参数和 SSA 变量              | 每个 block 可以接收参数                               |
| 支持控制流                       | 所有控制流如循环、if、跳转，都必须以 block 为单位组织 |
| 每个 Region 必须含至少一个 block | Region 中没有 block 是非法的 IR                       |



#### 5、MLIR强调的显示控制流和数据流

**MLIR 是一种“中间表示语言”，强调显式控制流和数据流，所以需要用 block 明确表示每一步的执行流程。**
 而像 C++ 这种高级语言，它把这些流程都“藏”起来了，由编译器帮你转换成底层的 block、跳转等结构。

**（1）C++ 是高级语言，**控制流是“隐式”的****

```c++
int sum_up_to_n(int n) {
    int sum = 0;
    for (int i = 0; i < n; i++) {
        sum += i;
    }
    return sum;
}

```

从这段代码看不到任何 “跳转”、“块” 的概念，对不对？

但实际上：**编译器会把它编译成 LLVM IR（或 MLIR），里面是明确的 block 和跳转！**

**（2）MLIR 是中间表示语言，控制流是“显式”, 必须写block**



#### 6、scf `Structured Control Flow Dialect`控制流方言

是 MLIR 提供的一个核心 Dialect，用来表达**结构化控制流结构**，比如：

- `scf.for`：结构化 for 循环
- `scf.while`：结构化 while 循环
- `scf.if`：结构化条件判断（if-else）
- `scf.yield`：结构化 Region 返回值

**（1）什么是结构化控制流**

控制流结构必须有**明确的入口和出口**，流程是**清晰、封闭、有层次**的，**不能随意跳转**。

举例：**其实早就会了，就是我们写c++、java等语言的模式，就是顺序+分支+循环**

```c++
if (a > b) {
  doSomething();
} else {
  doSomethingElse();
}
```

 有明确入口（if 判断）

 有结构闭合（if-else）

 执行流程是线性的（从 if 进入，最终执行完其中一分支，然后继续向下执行）



**非结构化语句：在底层的机器代码或中间表示中不一定按照结构化，会跳转**

```c++
int x = 5;

if (x > 0) goto label1;
x = x - 1;
goto end;

label1:
x = x + 1;

end:
std::cout << x << std::endl;

```

这里你看到了：

- 使用了 `goto` 可以**随时跳到另一个位置**
- 这种“跳转”会打破原有结构，程序流不再像块状结构，而是像箭头“乱飞”

**为什么要了解非结构化？我们一般不写**

在**编译器里**，你必须要理解：

- **机器指令是非结构化的！**
	 比如 x86 汇编中大量使用 `jmp`, `je`, `jne`, `call` 等跳转指令
- **LLVM IR（低级 IR）是非结构化的！**
	 它使用 `br`, `condbr` 等跳转

- **控制流图（CFG）会因为这些跳转很复杂！**
	 所以我们需要 MLIR 的 `scf` 这种“结构化表示”来暂时简化优化工作

**mlir中既有结构化代码，也有非结构化代码**

**结构化控制流**：
 MLIR 提供了 `scf` Dialect（Structured Control Flow）来专门表示结构化控制流语义，比如 `scf.for`、`scf.if`、`scf.while` 等。
 这部分代码看起来像高级语言里的循环和条件判断，**容易理解和优化**。

**非结构化控制流**：
 MLIR 也有专门的控制流 Dialect（`cf` Dialect），它提供了类似于跳转指令的操作，比如 `cf.br`（无条件跳转）、`cf.cond_br`（条件跳转），这就是非结构化控制流的表现形式。
 这种控制流对应的是类似汇编里跳转的指令，**更加底层，灵活但复杂**。



**（4）控制流图CFG**

**控制流图**就是描述这些代码块之间“执行顺序”或者“跳转关系”的图。

**在MLIR中的体现**

- MLIR 中的 **Block（基本块）** 是 CFG 的节点
- Block 里是一系列操作（Ops）
- 不同 Block 通过控制流操作（如 `cf.br`, `cf.cond_br`，或者结构化的 `scf.for` 内部隐式跳转）连接起来，形成边

比如你写的这个 `scf.for`：

```
scf.for %i = %zero to %n step %one {
  ...
}
```

这个例子内部可以被编译器展开成多个 Block，形成循环的 CFG。

**控制流图（CFG）在编译器中是真实存在的数据结构，图结构存储**
 但它也**确实可以被我们人类想象成一个图形**，来帮助理解程序的流程。

#### 在 MLIR 里：

- 每个 `Region` 有若干 `Block`，这些 `Block` 之间的跳转（如 `cf.br`、`cf.cond_br`）就构成了 CFG。
- MLIR 在 `mlir::Region`, `mlir::Block`, `mlir::Operation` 等 C++ 类中，**真实存储着控制流信息**。

**（3）**`scf.for`：这些本质上是Op， 而且他有循环体region

```mlir
%result = "scf.for"(%lower, %upper, %step) ({
  ^bb_loop(%i: index, %iter_args...):
    ...  // loop body
    "scf.yield"(%new_iter_vals) : (types) -> ()
}) {iter_args = [%init_val1, %init_val2, ...]} : (index, index, index) -> (result types)
```

示例代码

```mlir
%zero = "arith.constant"() {value = 0 : i32} : () -> i32
%one = "arith.constant"() {value = 1 : i32} : () -> i32

%result = "scf.for"(%zero, %n, %one) ({
  ^bb_loop(%i: i32, %sum: i32):
    %new_sum = "arith.addi"(%sum, %i) : (i32, i32) -> i32
    "scf.yield"(%new_sum) : (i32) -> ()
}) {iter_args = [%zero]} : (i32, i32, i32) -> i32
```

- `"scf.for"` 是 Op 的 **全名**（包括 Dialect）属于 `scf` Dialect（结构化控制流）

- 操作数（Operands）**起始值**：`%zero`（i = 0）**终止值**：`%n`（i < n）**步长**：`%one`（i += 1）

- `scf.for` 有一个 **region**，就是 loop 的循环体。region 里是一个 **block**（你可以命名为 `^bb_loop`），它有两个 block 参数：`%i`：当前循环迭代的 index（从 lower 到 upper，每步 step）`%sum`：你通过 iter_args 传进来的循环变量（迭代状态）
- 属性：`{iter_args = [%zero]}`，这个表示sum的初始值是`%zero`
- .yield则是把每次更新的sum返回去作为新的sum

在 block 内你可以写任何 Op，最后必须用 `"scf.yield"` 返回迭代值。

**（4）`scf.yield`是 `scf` 中循环或条件结构用来**返回值**的 terminator。**

也就是`iter_args`属性可以代表初始值，然后通过yield返回



**（5）为什么要这么复杂写一个for?**

### 因为 MLIR 是编译中间表示，它必须要：

1. **语义清晰明确**（谁是循环变量，谁是状态变量）
2. **控制流显式**（不能靠 `{}` 隐式划分作用域）
3. **支持 SSA（Static Single Assignment）模型**

举个例子，C++ 的 `sum += i` 是在原地改变量，在 MLIR 里是不允许的。

所以用 `iter_args = [...]` 和 `scf.yield` 明确地在每次循环之间传递新值。



#### 7、语法糖、原始语法、最原始语法

这里用的`scf.for`等依然是带有一定语法糖的Op版本，被叫做结构化Op，谈遏制上还是会变成原始的控制流Op，也就是自己写跳转的那种

```
scf.for     →    cf.cond_br + cf.br（lowering）
(结构化Op)        (原始控制流Op)
MLIR 会在后端优化或转换阶段，把 scf.for 自动变成 cf.br 控制流。
```

**（1）我写的是相对统一化写法的mlirOP**

MLIR 是基于 **Operation (Op) 为中心** 的中间表示，每一个 Op 都要遵循一种通用的统一语法结构：

```
mlir复制代码%result = "dialect.opname" (operands) {
  [optional blocks]
} {attributes} : (input types) -> (result types)
```

这就是所谓的 MLIR 的“统一 IR 语法”，是最正宗的写法。

示例：arith.addi

```
%3 = "arith.addi"(%1, %2) : (i32, i32) -> i32
```

- `"arith.addi"` 是 Op 名字（带有 Dialect 前缀）
- `(%1, %2)` 是输入参数
- `: (i32, i32) -> i32` 是类型签名
- 没有 attribute 和 block 就省略掉

你使用的 `"scf.for"`、`"func.func"`、`"arith.constant"` 等写法，全部符合这个模式。

2. 高层语法糖的“非统一性”，但是容易理解，与c++比较类似

高层语法糖，如：

```
mlir复制代码func.func @main(%arg0: i32) -> i32 {
  %0 = arith.constant 0 : i32
  ...
  return %0 : i32
}
```

虽然**更接近人类语言**，但从形式上来看它：

- 不再用 `"dialect.op"` 的格式
- 没有 `:` 后的 `(in) -> (out)` 明确表示
- attribute、regions、block等都被“隐藏”或格式变化



