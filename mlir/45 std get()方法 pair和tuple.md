#### 引

之前我们学习过`std::pair`，知道它可以通过`p.first, p.second`这样的方式取出两个元素，但是有一个更通用的方法，适用于`std::pair和std::tuple`这两种，比较通用

```c++
auto color = std::get<0>(attributes_map[pixel_format]);
```



#### 1、`std::get()`方法

这是一个c++ stl标准库的一个方法，适用于`pair/tuple`，可以指定取出第几个元素，对于`pair`来说只有0和1代表第一个和第二个元素，但是对于tuple来说则由tuple的长度决定

**语法结构：**

```c++
std::get<i>(tupleOrPair_variable)
```

模板参数 `<i>` 表示要取第几个元素（从 **0** 开始计数）；

参数 `tuple_variable` 是一个 `std::tuple或pair` 类型的变量；

返回值类型是该元素的类型。

**举例：**

```c++
#include <tuple>
#include <iostream>
using namespace std;

int main() {
    std::tuple<int, std::string, float> t = {42, "hello", 3.14f};

    cout << std::get<0>(t) << endl;  // 取第0个元素 -> 42
    cout << std::get<1>(t) << endl;  // 取第1个元素 -> "hello"
    cout << std::get<2>(t) << endl;  // 取第2个元素 -> 3.14
}

```

输出：

```
42
hello
3.14
```

#### 2  对比`pair`和`tuple`

## 

- `std::pair` 只有两个元素（`first`、`second`），适合简单场景；
- `std::tuple` 可以有任意多个元素（通过 `std::get<i>` 访问），适合复杂结构。

**举例：**

```c++
std::map<std::string, std::pair<int, int>> pair_map;
std::map<std::string, std::tuple<int, int, std::string>> tuple_map;
```

