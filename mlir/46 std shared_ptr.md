#### 1 `shared_ptr`

**智能指针模板类**   相较于传统在堆上创建对象用这个智能指针可以自动管理，不用我们手动删除

```c++
auto ptr = std::make_shared<int>(42);
```



#### 2、相较于直接创建指针的好处

```c++
int *p = new int(10);
```

手动在堆上创建，也要手动管理

那么使用这个智能指针就可以在最后一个引用使用完，离开作用域的时候自动删除

```c++
#include <memory>
#include <iostream>

int main() {
    std::shared_ptr<int> p1 = std::make_shared<int>(10);
    std::cout << *p1 << std::endl;  

    {
        std::shared_ptr<int> p2 = p1;  
        std::cout << "引用计数：" << p1.use_count() << std::endl;  
    } 

    std::cout << "引用计数：" << p1.use_count() << std::endl; 
} /

```

