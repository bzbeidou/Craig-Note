```c++
    auto returnOp = dyn_cast<ReturnOp>(entryBlock.back()).getOperation();
```

```c++
Operation *returnOp = entryBlock.back(); 
```

#### 1、种判断

| 函数              | 作用     | 失败时行为              |
| ----------------- | -------- | ----------------------- |
| `isa<T>(obj)`     | 是否为 T | 返回 `true` / `false`   |
| `cast<T>(obj)     | 转换为 T | 会 **断言失败（崩溃）** |
| `dyn_cast<T>(obj) | 转换为 T | 返回 `nullptr`          |

#### 2、类型转换

```c++
#include <iostream>
using namespace std;

class Animal {
public:
    virtual void speak() { cout << "Animal speaking\n"; }
};

class Dog : public Animal {
public:
    void speak() override { cout << "Dog barking!\n"; }
    void wagTail() { cout << "Dog wagging tail!\n"; }
};

```

```c++
Animal* a = new Dog();
a->speak();  
a->wagTail(); 
```

运行时类型转换呢

调用了一个函数`dynamic_cast`,返回了一个`Dog* d`

运行时进行的，运行时dynamic_cast函数会将输入尝试转换成泛型里`Dog*`的类型，如果不能转换就返回nullptr，如果转换成功了就直接返回

```c++
Animal* a = new Dog();

Dog* d = dynamic_cast<Dog*>(a); 
if (d) {
    d->wagTail();  
}
```

这里编译当然能通过，但是运行时一旦a本身并不是Dog的对象就完了

```c++
Dog* d = static_cast<Dog*>(a);
d->wagTail(); 
```

