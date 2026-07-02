#### 1、c++

**访问类的静态方法或变量**

```c++
class MathUtil {
public:
    static int square(int x) { return x * x; }
};

int a = MathUtil::square(5); // 使用 ::

```

```c++
std::vector<int> v; 
std::cout << "Hello"; 
```

```c++
//访问对象的成员函数或成员变量
std::vector<int> v;//创建了一个对象v
v.push_back(10);     // 使用 .访问实例对象的方法
int size = v.size(); // 使用 .

```

```c++
int value = 1;

class Test {
public:
    static int value;
};

int Test::value = 2;

int main() {
    std::cout << value << "\n";      // 全局变量 1
    std::cout << Test::value << "\n"; // 类静态变量 2
}

```

**（2）c里的结构体复习**

在 C 语言中，`struct` 是用来定义**一组相关数据的集合**，没有方法，只包含属性

```c
struct Point {
    int x;
    int y;
};
//使用
struct Point p1;
p1.x = 10;
p1.y = 20;

```

**为什么c++要引入类    -----实现对象方法、继承、多态等**

**（3）c++中的类   ----相对于java提供了更多权限，比如操作地址等**

**1）类定义对比**

```c++
#include <iostream>
#include <string>

class Person {
public:
    std::string name;

private:
    int age;

public:
    // 构造函数
    Person(std::string n, int a) {
        name = n;
        age = a;
    }

    void greet() {
        std::cout << "Hello, I am " << name << std::endl;
    }
};

```

**对比java，几乎一致**

```java
public class Person {
    public String name;
    private int age;

    public Person(String n, int a) {
        name = n;
        age = a;
    }

    public void greet() {
        System.out.println("Hello, I am " + name);
    }
}

```

**2）对象创建对比   Java 的对象都是在**堆上分配**，C++ 可以选择**栈 or 堆**，更灵活但更危险。** 

**c++有两种方式创建对象**

```c++
Person p("Tom", 18); // 栈上对象，自动析构（更常用）

Person* ptr = new Person("Tom", 18); // 堆上对象，需要手动 delete
delete ptr;

```

**java**

```java
Person p = new Person("Tom", 18); // 分配在堆上，GC自动回收
```

**3)构造函数 / 析构函数**

| 项目     | Java                    | C++                          |
| -------- | ----------------------- | ---------------------------- |
| 构造函数 | `public ClassName(...)` | `ClassName(...)`（无返回值） |
| 析构函数 | 没有（由 GC 管理）      | `~ClassName()`（类名加 ~）   |

C++ 析构函数在对象销毁时自动调用，非常重要，用于释放资源（文件、内存、句柄等）

**4）成员函数**

| 项目     | Java           | C++                               |
| -------- | -------------- | --------------------------------- |
| 访问方式 | `obj.method()` | `obj.method()` 或 `ptr->method()` |

**5)静态成员(类变量和类方法)**

| 特性   | Java                 | C++                   |
| ------ | -------------------- | --------------------- |
| 声明   | `static` 关键字      | `static` 关键字       |
| 调用   | `ClassName.method()` | `ClassName::method()` |
| 初始化 | 定义时直接赋值       | 类外单独初始化变量    |

**6）继承和多态**

**c++**C++ 需要用 `virtual` 关键字才能实现“多态”，Java 自动支持

```c++
class Animal {
public:
    virtual void speak() { std::cout << "..." << std::endl; } // 注意 virtual
};

class Dog : public Animal {
public:
    void speak() override { std::cout << "Woof!" << std::endl; }
};

```

**java**

```java
class Animal {
    public void speak() { System.out.println("..."); }
}

class Dog extends Animal {
    public void speak() { System.out.println("Woof!"); }
}

```



#### 2、深度学习分类  常见的由有CV、NLP等

| 应用领域                   | 典型任务                     | 模型示例             |
| -------------------------- | ---------------------------- | -------------------- |
| **图像处理（CV）**         | 图像分类、目标检测、人脸识别 | LeNet、ResNet、YOLO  |
| **语音处理（ASR）**        | 语音识别、语音合成           | DeepSpeech、Tacotron |
| **自然语言处理（NLP）**    | 机器翻译、对话生成、文本分类 | BERT、GPT、LLaMA     |
| **多模态（图文、图声等）** | 看图说话、语音+文字输入      | CLIP、Gemini、GPT-4V |
| **强化学习（RL）**         | 玩游戏、机器人控制           | DQN、PPO、AlphaGo    |

常见的手写数字识别就是CV的经典例子，用的结构是卷积神经网络CNN。但是大语言模型是属于NLP方向，用的是Transformer架构而不是CNN。



#### 3、Transformer架构

**Transformer机构很重要，是最近几年深度学习领域的重要突破之一**

**（1）Transformer架构长什么样？  ---由两部分组成**

编码器+解码器

对大语言模型来说，通常只用解码器Decoder部分

```
输入 → [编码器 Encoder × N层] → 中间表示 → [解码器 Decoder × N层] → 输出
```

针对Decoder部分可以分为以下几个部分：

```
输入词向量 + 位置信息
↓
多头注意力机制（看上下文）
↓
前馈网络（做理解）
↓
N 层堆叠（越多越深）
↓
输出结果（预测下一个词 or 分类）

```

