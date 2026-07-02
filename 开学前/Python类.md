# python 补充知识

## 1 类定义

```py
class BaseConverter(object):
```

定义一个类，（）里的内容是他继承python自带的object类，默认写不写都行

**对比java**

| 功能/概念          | Java 中的写法                   | Python 中的写法                       | 差异说明                            |
| ------------------ | ------------------------------- | ------------------------------------- | ----------------------------------- |
| 定义类             | `class MyClass { ... }`         | `class MyClass:`                      | 不写 `{}`，用缩进                   |
| 构造函数（初始化） | `public MyClass(...) {}`        | `def __init__(self, ...):`            | Python 叫 `__init__`，必须用 `self` |
| 创建对象           | `MyClass obj = new MyClass();`  | `obj = MyClass()`                     | Python 没有 `new`                   |
| 成员变量访问       | `obj.name`（需要先定义 `name`） | `obj.name`                            | 都是 `.` 调用，Python 变量更自由    |
| 继承               | `class Dog extends Animal`      | `class Dog(Animal):`                  | Python 用括号直接写父类             |
| 接口/抽象类        | `interface` / `abstract class`  | `from abc import ABC, abstractmethod` | Python 用 `abc` 模块支持接口/抽象   |

**主要区别在于python的成员变量**

Python 成员变量**不需要提前声明**，直接赋值就可以

```py
class Person:
    def __init__(self, name):
        self.name = name  # 直接赋值就定义了这个变量

```

`self.name`：就是这个对象自己的成员变量。

没有类型声明、也不需要提前写。

赋值时自动“声明 + 初始化”。

**Python 中没有 Java 那样严格的 `public` / `private` / `protected` 关键字，但有“约定俗成的命名规则”。**

| 写法     | 含义（对应 Java）       | 用法说明                                        |
| -------- | ----------------------- | ----------------------------------------------- |
| `name`   | 公开变量（public）      | 所有人都能访问                                  |
| `_name`  | 受保护变量（protected） | 约定给类自己或子类使用，**只是建议**，不会强制  |
| `__name` | 私有变量（private）     | Python 会做“名字改写”避免外部访问（不完全私有） |

```python
class Person:
    def __init__(self):
        self.name = "Tom"       # public
        self._age = 30          # protected（其实别人也能访问）
        self.__password = "123" # private（别人不能直接访问）
```

```py
p = Person()
print(p.name)       # ✅ 可以访问
print(p._age)       # ⚠️ 虽然能访问，但不推荐（受保护变量）
print(p.__password) # ❌ 会报错 AttributeError

```





## 2 python类继承机制

| 功能       | Java                       | Python                 |
| ---------- | -------------------------- | ---------------------- |
| 定义继承   | `class Dog extends Animal` | `class Dog(Animal):`   |
| 调用父构造 | `super()`                  | `super().__init__()`   |
| 方法重写   | `@Override` 注解           | 不需要写注解，直接覆盖 |



```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        print(self.name + " makes a sound.")

class Cat(Animal):
    def __init__(self, name, color):
        super().__init__(name)  # 调用父类构造函数
        self.color = color

    def speak(self):
        print(self.name + " the " + self.color + " cat says: Meow!")

c = Cat("Kitty", "white")
c.speak()
```

```
Kitty the white cat says: Meow!
```





## 3 python如何定义一个类

看前面

## 4 类的初始化函数以及成员变量

```py
    def __init__(self, no_save: bool = False):
```

`__init__` 是 Python 类的特殊函数，**创造这个类的对象时自动运行**

`self` 是“当前这个对象自己”（每个对象都有自己的工具，不会和别人混）。

`no_save: bool = False` 是一个参数：

- `no_save` 是一个布尔值参数（只能是 `True` 或 `False`）
- 默认是 `False`（就是说默认会保存中间文件）



**java中初始化函数用类名**





**成员变量**

```py
        self.operands = dict()
        self.tensors = dict()
        self.shapes = dict()
        self.input_names = list()
        self.output_names = list()
```

| 变量名              | 类型         | 说明                                      |
| ------------------- | ------------ | ----------------------------------------- |
| `self.operands`     | 字典（dict） | 存放操作数                                |
| `self.tensors`      | 字典         | 存放张量信息                              |
| `self.shapes`       | 字典         | 存放每个数据的形状，比如 (1, 3, 224, 224) |
| `self.input_names`  | 列表（list） | 存放输入的名字                            |
| `self.output_names` | 列表         | 存放输出的名字                            |

**与java的区别在于python的成员变量支持直接访问，但是java是建议封装使用的**

| 功能         | Java                  | Python                         |
| ------------ | --------------------- | ------------------------------ |
| 定义变量位置 | 类体中、通常 private  | 构造函数中 `self.` 直接定义    |
| 访问方式     | 用 getter/setter      | 可以用方法，也可以直接访问     |
| 是否强制封装 | 是（推荐私有 + 方法） | 否（可以直接访问 `obj.score`） |
| 是否强类型   | 是（需指定类型）      | 否（动态类型，运行时判断）     |

## 5 字典dict、列表list

为了存储很多数据，python有两种容器来存储内容

`list`：用来装“有序的数据集合”

`dict`：用来装“带标签的数据集合”

**list**

`list` 就像是一个**可以装很多东西的篮子**，里面的内容是**按顺序排列的**，可以重复，可以随时加、删、改。

python中常用list，list就是相当于python中的数组，但是也会有常用的第三方库数组，比如numpy.array这种

**（1）创建一个list**

```py
fruits = ["apple", "banana", "orange"]
```

**（2）访问元素（从0开始编号）**

```py
print(fruits[0])  # 输出 apple
print(fruits[1])  # 输出 banana
```

**（3）修改、添加、删除元素**

```py
fruits[1] = "pear"       # 把 banana 改成 pear
fruits.append("grape")   # 在末尾添加 grape
fruits.remove("apple")   # 删除 apple
```



| 特点           | 描述                                |
| -------------- | ----------------------------------- |
| 有序           | 有位置编号，从 0 开始               |
| 可重复         | 可以放两个一样的值                  |
| 可变（可修改） | 能增删改查                          |
| 类型不限       | 可以放字符串、数字、甚至另一个 list |



**dict**

`dict` 就像是**一张表、一张字典、一个电话号码簿**，是由一对一对的 `key:value` 键值对组成的，没有顺序，但可以通过 `key` 快速找到对应的 `value`。

**（1）创建一个dict**

```py
scores = {
    "Tom": 90,
    "Lily": 85,
    "Jack": 95
}

```

**（2）访问value（通过key访问）**

```py
print(scores["Tom"])  # 输出 90

```

**（3）添加、修改、删除键值对**

```py
scores["Mary"] = 88        # 添加
scores["Tom"] = 93         # 修改
del scores["Lily"]         # 删除 Lily 这一项

```

| 特点       | 描述                                       |
| ---------- | ------------------------------------------ |
| 键值对结构 | key 是名字/编号，value 是值                |
| 无序       | 不按顺序排列（Python 3.7+ 会保留插入顺序） |
| key 不重复 | 同一个 key 只能对应一个 value              |
| 查找速度快 | dict 是哈希结构，查找非常快                |

**实例**

```py
students = ["Tom", "Lily", "Jack"]
scores = {"Tom": 90, "Lily": 85, "Jack": 95}

for student in students:
    print(f"{student} 的分数是 {scores[student]}")

```

