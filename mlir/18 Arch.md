```c++
  template <typename FPtrTy> FPtrTy CastToFPtr(const char *symbolName) {
    assert(DL.isValid());
    auto fPtr = DL.getAddressOfSymbol(symbolName);
    if (fPtr == nullptr) {
      llvm::errs() << "can't find symbol: " << symbolName << "\n";
      llvm_unreachable(symbolName);
    }
    return reinterpret_cast<FPtrTy>(fPtr);
  }
```

### 1、首FPt

参数是一个字符

通过字符串转

### 2、逻辑函数

```c++
extern "C" int add(int a, int b);
```

```cpp
#include "libmath.h"
int main() {
    int result = add(3, 4); 
}
```

```c++
using AddFuncTy = int(*)(int, int);
auto addFunc = CastToFPtr<AddFuncTy>("add");  
int result = addFunc(3, 4);                   
```

### 3、使用场景

- 头文件
- **函数**
- 已经加载到内存了

```c++
using FuncTy = int(*)(int, int);  
auto myAdd = CastToFPtr<FuncTy>("my_add"); 
int result = myAdd(1, 2);              
```
