#### 芯片底层的链接

### 1、不同的AI芯片

- 有 16 个 NPU，本地内存 512KB
- 有 32 个 NPU，本地内存 1MB

需要知道芯片的配置，从而可以：

（1）计算算子的开销

（2）判断模型能不能放得下

（3）调用正确的实现函数（不同芯片的算子实现可能不同，要从对应的动态库里加载函数指针）

### 2、

```c++
Y = Conv2D(X, W, stride=1, padding=1);
```

**（1）内存估算**

- 它需要多少片上内存 (LMEM)？
- **返回结果大于芯片的 `LMEM_BYTES`说明放不下需要切分计算。**

**（2）并行计算**

- 芯片16个NPU每个能一次处理8字节
- **`Arch::en_num(..)`**

**（3）调用函数**

```cpp
using conv_func_t = void(*)(float*, float*, float*);
auto conv = Arch::CastToFPtr<conv_func_t>("conv2d_kernel");
conv(X, W, Y);  
```

