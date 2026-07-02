### 1、变量

```c++
static int64_t NPU_NUM;         
static int64_t EU_BYTES;        
static int64_t LMEM_BYTES;     
static int64_t LMEM_BANKS;      
static int64_t LMEM_BANK_BYTES;
static llvm::StringRef LIB_BACKEND_NAME; 
static bool ALIGN_4N;           
static module::Chip chip;       
static uint64_t FREQ;           
```

### 2、静态

**（1）初始化**

```c++
static void init(uint64_t freq);   
static uint64_t get_frequance(); 
```

**（2）内存**

```c++
static int64_t eu_num(double dbytes);
static size_t get_gmem_bytes(Value v);
static int64_t get_tensor_lmem_bytes(...);
static int64_t get_weight_lmem_bytes(...);
```

一个张量占多少内存

权重占多少内存

Execution Unit 的数量如何换算

```c++
get_tensor_lmem_bytes(v, n, c, d, h, w, true)
```

**（3）解析 **

```c++
template <typename FPtrTy>
FPtrTy CastToFPtr(const char *symbolName);

template <typename FPtrTy>
FPtrTy PplCastToFPtr(const char *symbolName);

template <typename FPtrTy>
FPtrTy CastToDQFPtr(const char *libName, const char *symbolName);

template <typename FPtrTy>
FPtrTy CastToCustomFPtr(const char *symbolName, bool fatal = true);

template <typename FPtrTy>
FPtrTy CastToCustomPluginPtr(const char *symbolName);
```
