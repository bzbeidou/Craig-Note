

#### 1、关于系统自带的.td文件

建议掌握的文件内容

| 文件                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `OpBase.td`                                                  | 所有 dialect 定义的基础语法，必须懂（你自定义 op/attr/type 都依赖它） |
| `TypeBase.td`, `AttrBase.td`                                 | 自定义 Type 和 Attribute 的通用基类                          |
| `EnumAttr.td`                                                | 定义枚举型 Attribute 的工具（你已经在用了）                  |
| `mlir/include/mlir/IR/BuiltinOps.td`                         | 看看内置操作是怎么写的，能学会规范写法                       |
| 你自己的 `MyDialectOps.td`, `MyDialectTypes.td`, `MyDialectAttrs.td` | 写你自己的 dialect 就够用了！                                |



**（1）DialectBase.td**

**TableGen中的class   叫类模板，与类不同**

*三个要点*

*1.1 class是模板，不会生成具体地c++代码*

*1.2 class被别的类通过实现或继承：才能起作用，生成c++代码*

*1.3 class相当于tablegen里最底层的部分，所有其他部分都建立在class之上*



**详细分层，类似于建筑结构**

| 层级       | 类型                                 | 作用                   |
| ---------- | ------------------------------------ | ---------------------- |
| **最底层** | `class` 模板                         | 定义结构（字段、继承） |
| **中间层** | 继承模板的 `class`（有时也生成代码） | 构造更具体描述结构     |
| **顶层**   | `def` 声明（通常用于代码生成）       | 真正生成 C++ 代码      |

来自DialectBase.td的实际例子

```
//定义一个Dialect的class模板
class Dialect{
	string name = ?;
	string cppNamespace = ?;
}
```

```
// 使用模板定义一个特定 Dialect
def Toy_Dialect : Dialect {
  let name = "toy";
  let cppNamespace = "::mlir::toy";
}
```



Dialect部分的层级结构可以认为最底层就是Dialect，我们自己定义Dialect也是从最底层开始的



**（2）EnumAttr.td文件**

*以两个部分的代码为例*

```
class EnumCase<string sym, int intVal, string strVal, int widthVal> {
  string symbol = sym;
  int value = intVal;
  string str = strVal;
  int width = widthVal;
}

class IntEnumAttrCaseBase<I intType, string sym, string strVal, int intVal> :
    EnumCase<sym, intVal, strVal, intType.bitwidth>,
    SignlessIntegerAttrBase<intType, "case " # strVal> {
  let predicate =
    CPred<"::llvm::cast<::mlir::IntegerAttr>($_self).getInt() == " # intVal>;
}

```

**这两个都是class模板类，只不过第二个继承了第一个，而且在第一个class的基础上还新增了一个字段，但是这两个都不会生成c++代码，只有写出def来定义枚举的每一项时才会去生成c++代码**

**只不过第二个class在第一个基础上做了一些限定，针对某一类型做枚举属性**



定义Attribute时的层级

| 层级     | 类名                    | 作用                                           | 是否最底层                     |
| -------- | ----------------------- | ---------------------------------------------- | ------------------------------ |
| 最高层   | `EnumAttr`              | 定义 Enum 类型的 Attribute                     | ❌                              |
| 中间层   | `AttrDef`               | 通用 Attribute 定义模板（接收 Dialect/类名等） | ✅（通常视为 Attribute 的底层） |
| 底层工具 | `Attr` / `TypeAttrBase` | 更低层基础类，用于匹配 Attr 类型和存储实现细节 | ✅✅（更偏向基础库）             |

                            +----------------+
                            |    AttrDef     |
                            +--------+-------+
                                     ^
                                     |
              +----------------------|----------------------+
              |                     |                      |
       +-------------+     +----------------+     +----------------+
       |  EnumAttr   |     |  StructAttr    |     |  CustomAttr    |
       +-------------+     +----------------+     +----------------+





**（3）在.td内的传参机制  从class 到 class继承 再到 def实现过程中的传参**

*第一 定义基础模板类   也可以直接看自带的.td文件，不如EnumAttr里很多*

```
// 基础模板类，相当于 EnumCase
class BaseCase<string symbol, int value> {
  string name = symbol;
  int val = value;
}
//很明显定义了两个字段name和val，而且是通过传参获取
```

*第二 定义一个继承它的类，传入自己的模板参数*

```
// 继承类
class DerivedCase<string sym, int v> : BaseCase<sym, v> {
  string extraInfo = "This is derived.";
}

//这里DerivedCase把自己接收到的sym和v，传给继承的BaseCase，实际上就和c++继承一样的，相当于继承了public成员变量。同时这里还新增了一个自己的字段
```

*第三 用def实例化最终结构*

```
def MyItem : DerivedCase<"Add", 1>;

这里DerivedCase<"Add", 1>被实例化，同时把参数传给BaseCase

最终的MyItem拥有以下信息
MyItem {
  name = "Add";         // 来自 BaseCase
  val = 1;              // 来自 BaseCase
  extraInfo = "...";    // 来自 DerivedCase
}
```



**（4）Type层级结构示意**

| 类名      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| `TypeDef` | 所有 Type 的通用模板类（类似于 Attribute 中的 `AttrDef`，继承自Type） |
| `Type`    | 更底层的父类，封装了一些 Trait、类型匹配等                   |



实际上在还有继承`TypeDef`的模板类，但是我们自己定义Type的时候常用`TypeDef`，所以这里不讲



#### 2、写一个mlir文件看看attr怎么用的

*假设test.mlir文件如下*

```mlir
module {
  "test.dummy_op"() { my_enum = #my<"A"> } : () -> ()
}
```

其中：

- `"test.dummy_op"` 是一个虚构的操作，我们没有定义它
- `{ my_enum = #my<"A"> }` 表示它有一个 Attribute，叫 `my_enum`，它使用了你自己定义的 EnumAttr
- `#my<"A">` 这一部分就是 MLIR 的 “EnumAttr 文字量” 写法（直接使用 EnumAttr）

*当运行如下部分时*

```
mlir-opt test.mlir
```

这时mlir会解析这个IR，解析定义的Dialect和Attribute，然后检查语法是否正确，格式化并且重新打印IR



*典型输出结果如下：*

```
module {
  "test.dummy_op"() {my_enum = #my<"A">} : () -> ()
}
```

这里明显和输入内容是一样的，这就说明MLIR正确是必然了EnumAttr(#my<"A">)并成功解析了整个IR文件



*如果写错了，如下*

```
module {
  "test.dummy_op"() {my_enum = #my<"Z">} : () -> ()
}

```

那么不会正常打印，因为MLIR 是**真正在使用你定义的 EnumAttr 的信息进行校验的**



*补充：怎样让mlir-opt知道自定义的EnumAttr----需要注册Dialect、EnumAttr*

```c++
int main(int argc, char **argv) {
  mlir::DialectRegistry registry;
  registry.insert<mlir::my::MyDialect>(); // 插入你的 dialect

  mlir::registerAllDialects(registry); // 插入 builtin 的
  mlir::MLIRContext context(registry);

  return mlir::asMainReturnCode(
    mlir::MlirOptMain(argc, argv, "Test MyEnumAttr", context));
}

```





#### 3、mlir

以这个例子进行解析,这里面的各个句子都是什么意思？

```
module {
  "test.dummy_op"() {my_enum = #my<"Z">} : () -> ()
}

```

**3.1整体层次**

```
module {
  └── "test.dummy_op"             // 一个操作（Operation）
        ()                        // 操作的输入（operand）
        {my_enum = #my<"Z">}      // 操作的 attribute 字典
        : () -> ()                // 操作的输入类型和输出类型（Function Type）
}
```

**3.2 module { ... }**

MLIR 中最顶层的容器

**3.3` "test.dummy_op"` 是什么**

这是mlir中自定义操作Operation，名字的结构为

```
"dialect_name.operation_name"
```

- `test` 是 Dialect 名称（就是你定义的 dialect，比如 `MyDialect`）
- `dummy_op` 是 Op 名字（这里我们只是为了测试写的占位符）

即使你没定义这个 Op，也可以写，只要语法合法，MLIR 会把它当成一个“未知操作”处理。

**3.4  `()` 是什么意思？**

这是 **操作的输入 operand 列表**，在 MLIR 中，每个操作可以接受一组输入值，比如：

```
%out = "my.add"(%a, %b) : (tensor<f32>, tensor<f32>) -> tensor<f32>

```

这里`%a`, `%b` 就是两个 operand，在我的例子里`()`表示这个操作不接受任何输入

**3.4`{ my_enum = #my<"Z"> }`是该操作的属性字典**

| 部分       | 解释                                                 |
| ---------- | ---------------------------------------------------- |
| `my_enum`  | 是属性（Attribute）的名字，就像变量名一样，取名随意  |
| `=`        | 赋值符号                                             |
| `#my<"Z">` | 是 MLIR 中 **EnumAttr 的字面量语法**（后面详细解释） |

**3.5`#my<"Z">` 是什么**

这是一个具体的写法

```
#<dialect_name><"enum_case_string">
```



#### 4、c++

**（1）哈希表  HashTable   一种数据结构   就是数组+哈希函数**

*数组的每个位置叫做bucket桶，桶里就可以放键值对*

目的：数组查找数据需要遍历，使用哈希表可以快速定位

对数组来说

```c++
["苹果","香蕉","橘子"。。。]//名称（键）
["2","3","4"...]//价格（值）
//这些,比如我想找西瓜价格，那么只能遍历，因为我不知道西瓜的索引，
//所以考虑到如果提前制定规则知道索引就好了------哈希函数即是规则
```

*哈希函数*

定义一个函数，输入是存储的内容，然后对其经过一定的计算得到一个数，这个数是固定的，然后放到数组这个数的位置

```c++
hash("苹果") = 3
hash("香蕉") = 7
//这个计算是由一定规则的，比如把字母转换成ASCII码相加得到数值
//那么就把对应的价格放到对应编号的数组里了
```

*优势*

查找的时候可以输入需要查找的键，也就是”西瓜“，通过哈希函数就可以得到存放它价格的数组索引，这样就能直接找到价格



*问题：哈希冲突   哈希函数通过键计算索引可能会相同*

比如

```c++
hash("Tom") = 4
hash("Jim") = 4
//这两个都是通过ASCII码相加得到的
```

这种可以用链表的方式存储



**（2）复杂度  ----衡量程序运行效率**

*时间复杂度：描述程序执行所需步骤*

*空间复杂度：描述程序需要用多少内存*

用O表示，指Order数量级的意思

- `O(1)`：固定步骤，一步就能完成操作。**最理想**
- `O(n)`：，线性增长，随着输入变多，步骤数也线性增加
- `O(n^2)`：每多一个元素，操作数增加得更多（平方增长）

*常见复杂度等级，从快到慢*

| 复杂度     | 名称         | 举例说明                                      |
| ---------- | ------------ | --------------------------------------------- |
| O(1)       | 常数时间     | 查数组中第一个元素 `arr[0]`，永远只需1步      |
| O(log n)   | 对数时间     | 二分查找，比如猜一个1-100的数字，每次减半范围 |
| O(n)       | 线性时间     | 遍历数组，挨个访问                            |
| O(n log n) | 线性对数时间 | 高效排序算法（归并排序、快速排序）            |
| O(n²)      | 平方时间     | 嵌套两层循环，比如冒泡排序                    |
| O(2ⁿ)      | 指数时间     | 穷举所有子集，比如暴力解子集和问题            |

**（3）c++的unordered_map   无序表**

**底层就是哈希表**

这个和map的区别

| 特性          | `map`                 | `unordered_map`            |
| ------------- | --------------------- | -------------------------- |
| 底层结构      | 红黑树（二叉搜索树）  | 哈希表                     |
| 是否有序      | 是，按照 key 升序排序 | 否，key 是无序的           |
| 查找/插入效率 | O(log n)              | O(1)（平均），O(n)（最坏） |
| 适合场景      | 需要顺序遍历的情况    | 只关注查找速度的情况       |

**基本语法和使用**

```c++
#include <unordered_map>
std::unordered_map<std::string, int> wordCount;
//这里键是string，值是int

//插入元素
wordCount["apple"] = 3;
wordCount.insert({"banana", 2});

//查找元素
std::cout << wordCount["apple"] << std::endl; // 输出 3

if (wordCount.find("pear") != wordCount.end()) {
    std::cout << "pear 存在" << std::endl;
}

//遍历元素
for (const auto& [key, value] : wordCount) {
    std::cout << key << " : " << value << std::endl;
}
```


