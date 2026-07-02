#### 1、clone的LLVM是什么

`llvm-project` 是一个**庞大的 C++ 编译器工具链集合**，包括多个模块和库，不是一个直接可以运行的“应用”。

**包含的项目有**下面这些等等

| 名称  | 说明                               |
| ----- | ---------------------------------- |
| LLVM  | 编译器基础架构库（IR、优化器等）   |
| Clang | C/C++ 编译器前端                   |
| MLIR  | 多层次中间表示，用于构建新的编译器 |
| LLD   | 链接器                             |
| LLDB  | 调试器                             |



**2、为什么下载好了llvm-project里面的mlir、llvm这些还需要构建呢？**

*因为下载的是源码，也就是全都是cpp文件，需要编译构建build，把他变成可以用的应用工具*

| 阶段       | 类比                                                        |
| ---------- | ----------------------------------------------------------- |
| clone      | 你去网上下载了一堆 Lego（源码）                             |
| cmake      | 你打开 Lego 说明书，决定要拼哪些模块（配置 build）          |
| ninja/make | 你真正动手拼装，最后拼成一个“飞机”或“城堡”（构建成工具/库） |

构建过程 ： clone -> cmake -> ninja/make

*构建后生成了什么，构建后一般放在/build文件夹内*

在你设置的 `build/` 目录下，会生成：

- 可执行程序（比如 `bin/mlir-opt`, `bin/clang`）
- 动态或静态链接库（比如 `lib/libMLIR.so`）
- 中间的对象文件、头文件、工具链配置等



**平常情况下Clang不是不用构建吗？**

你平常在系统里能直接用 `clang`，比如通过 `apt install clang` 安装的，这是构建好的版本。现在是从源码出发，要深入学习和修改它的功能，所以必须自己构建一套属于你自己的编译器工具链。



#### 2、C++

**（1）为什么有的不用.h**

c++标准库的头文件不带.h，自己写的头文件要带上.h

**（2）c++中传统的enum和enum class**

*注意：enum class相当于一种新的数据类型，不是enum和class的拼接*

*推荐用enum class，类似于一个小的命名空间*

```c++
enum class Token {
    identifier,
    number
};

Token tok = Token::identifier;
//这里很明显用的时候是在Token命名空间内的，也就是这个enum class就像一个迷你的命名空间,访问到了Token这个命名空间内的identifier这个枚举
```

```
enum Token {
    identifier,
    number
};

Token tok = identifier; // 这是传统写法命名污染，全局就有 identifier 了
```

*区别就是一个需要作用域前缀访问，另一个不需要*



**为什么identifier类型都没有，看起来很奇怪**

实际上identifier就是Token类型的，

```c++
Token tok = Token::identifier;
//这句代码很明显能说明这个问题，定义了一个Token类型的tok变量，然后将Token这个作用域内的identifier赋值给这个tok，第一个Token就是数据类型，指的枚举类型；第二个Token则是指的命名空间
```



在代码里有

```c++
enum class Token {
    eof = -1,
    def = -2,
    extern_ = -3,
    identifier = -4,
    number = -5,
    ...
};

//这里给枚举值都赋值了整数类型，本质上也就是整数，但是被封装在了Token里面避免了会出现重名的情况，使用时需要指定作用域Token
```

也就是

```c++
const int TOKEN_IDENTIFIER = -4;
//等同于
enum class Token {
    identifier = -4
};

```

**注意如果把枚举值转换成int值需要强制转换**

```c++
int val = static_cast<int>(tok); // ✅ 强制转换

```



#### 3、词法分析器Lexer做什么

*把toy语言内容按语法元素拆分成一个个Token单元*

```toy
def foo(x y)
  x + y * 2

```

*分词结果：*

| 单词/符号 | Token 类型     | 枚举值（你的定义）  | 备注说明                     |
| --------- | -------------- | ------------------- | ---------------------------- |
| `def`     | 函数定义关键字 | `Token::def`        | 说明一个函数的定义开始       |
| `foo`     | 标识符         | `Token::identifier` | 函数名                       |
| `(`       | 符号           | `'('`（ASCII符号）  | 不是枚举里特殊值，直接用字符 |
| `x`       | 标识符         | `Token::identifier` | 参数                         |
| `y`       | 标识符         | `Token::identifier` | 参数                         |
| `)`       | 符号           | `')'`（ASCII符号）  |                              |
| `x`       | 标识符         | `Token::identifier` | 变量                         |
| `+`       | 运算符         | `'+'`（ASCII符号）  |                              |
| `y`       | 标识符         | `Token::identifier` | 变量                         |
| `*`       | 运算符         | `'*'`（ASCII符号）  |                              |
| `2`       | 数字常量       | `Token::number`     | 数字值                       |

*处理流程*

词法分析器会逐字符扫描代码内容：

1. 遇到 `def`，判断它是关键字，返回 `Token::def`。
2. 接下来的 `foo` 是用户定义的名字 → `Token::identifier`。
3. 然后遇到 `(`，是符号 → 返回 `'('`。
4. `x` → `identifier`，`y` → `identifier`。
5. 直到 `)`，返回 `')'`。
6. 然后是表达式：`x + y * 2`，也按 identifier、符号和 number 拆解。

**词法分析器会传给语法分析器两个配合从而构造出AST**



**3、关于类里的定义细节部分**

**（1）函数后面跟了const:表示这个函数不能修改类里的任何成员变量**

```c++
//例：下面这个就是反例
class Test {
    int val = 42;
public:
    int getVal() const {
        val = 10;  // ❌ 错误！const 函数不能修改成员变量
        return val;
    }
};

```

**（2）.h内的函数的返回值问题**

```c++
//.h文件和.c文件很类似

Token getCurrentToken() const;//这种只声明，不需要写返回值

Token getCurrentToken() const {
    return currentToken;
};//这种直接定义了所以必须带上返回值，而且在c文件里也不能写了
```

**（3）关于::和.的问题**

**一句话：(包括.c定义.h这种)类外部定义类的函数时用::  通过实际创建的对象调用函数时用.或->**

```c++
class MyClass {
public:
    void sayHello(); // 函数声明
};

// 函数定义（在类外部）
void MyClass::sayHello() {
    std::cout << "Hello!" << std::endl;
}
//这种就是在.c内定义.h的情形

//调用则用.
MyClass obj;
obj.sayHello();  // 使用 .


```

**(4)初始化列表是什么？**

*如下，函数名后面加冒号然后带了一个input和index*

```c++
Lexer::Lexer(std::string input) : input(std::move(input)), index(0) {
    currentToken = getNextToken();
}

//input和index就是这个Lexer类的两个成员变量
//input()，index()后面括号里的内容就是对input成员变量进行复制，这里就是把参数的input传给了input成员变量，然后零index=0.
//实际上也可以直接在构造期内进行赋值，但是推荐用初始化列表

//等同于
Lexer::Lexer(std::string input) {
    this->input = std::move(input);//可以不用在意这个std::move
    index = 0;
    currentToken = getNextToken();
}

```

**（5）char问题**

下面这个明显得到的是char类型，为什么用int接收

```c++
int Lexer::getChar(){
    if(index >= input.size())
        return EOF;
    return input[index++];
}
//char类型本质就是int类型，但是有一个问题是char是无法确定正负的，有的计算机认为其是-128-127，有的则是0-255
//但是我们这里的EOF就是-1，如果用char可能接收不到，所以用int
//注意这里的EOF和input[index++]的值不会冲突，char的内容是ascii码，ascci码是0-127的范围，不会有-1，但是-1决定了用int接受

//EOF  是头文件里的一个宏定义，就是常数-1，代表end of file，这个文件内容结束了，就是读到头了
```

**（6）词法分析器怎么判断的identifier，就是自己定义的变量这种**

搭配ungetChar小函数

```c++
//这个获取字符并向下推了一位
int Lexer::getChar() {
    if(index >= input.size())
        return EOF;
    return input[index++];
}

//有时我们想回退，这就是判断identifier要用
void Lexer::ungetChar() {
    if(index > 0)
        --index;
}

```

```c++
举例
输入： "abc+123"

你在识别 "abc" 作为 identifier 的时候：
- 你读了 'a' -> 合法
- 再读 'b' -> 合法
- 再读 'c' -> 合法
- 再读 '+' -> 不合法！（不是字母/数字）

这时候你需要退回那个 '+'，让下一个阶段再处理它。
就调用ungetChar();  // 把 index--，让下次 getChar() 再返回 '+'
```

