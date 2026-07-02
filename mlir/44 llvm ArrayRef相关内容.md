

#### 1、轻量容器类

```c++
void printArray(llvm::ArrayRef<int64_t> arr) {
  for (auto x : arr)
    std::cout << x << " ";
  std::cout << std::endl;
}

int main() {
  std::vector<int64_t> v = {1, 2, 3};
  llvm::SmallVector<int64_t, 4> s = {4, 5, 6};
  int64_t a[] = {7, 8, 9};

  printArray(v);  // 
  printArray(s);  // 
  printArray(a);  // 
}

```

#### 2、内部结构

针对不同的类型的vector、array等做了判单，存入指针。

一个方面就是统一，另一个方面就是其中还存了长度

#### 3、本身

```c++
static void getNCHW_align_right(llvm::ArrayRef<int64_t> &shape, int64_t &n,
                                int64_t &c, int64_t &h, int64_t &w) {
```

**ArrayRef ，可以理解为引用，但是是存的指针**