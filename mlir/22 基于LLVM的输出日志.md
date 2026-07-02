#### 1 头文件

```cpp
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include <cstdarg>
#include <cstdio>
#include <iostream>
#include <memory>
#include <sstream>
#include <string>
```

#### 2 命名空间

```c++
namespace tpu_mlir {
extern int32_t cur_log_level;

```

只是声明，不分配内存。

控制当前的日志等级，例如：

-  表示输出所有日志；
- 只输出 Pass 级日志；
- 只输出 Layer Group 日志。

#### 3   函数

```cpp
inline std::string formatString(const char *format, ...) {
  va_list args;
  va_start(args, format);
  size_t size = vsnprintf(nullptr, 0, format, args) + 1;
  std::unique_ptr<char[]> buf(new char[size]);
  vsnprintf(buf.get(), size, format, args);
  va_end(args);
  return std::string(buf.get(), buf.get() + size - 1);
}
```

经常用到，比如：

```cpp
std::string msg = formatString("layer %d takes %.2f ms", layer_id, time);
```

#### 4  `PASS_LOG_DEBUG_BLOCK` 

```cpp
#define PASS_LOG_DEBUG_BLOCK(block) \
  do { \
    if (cur_log_level == 1 || cur_log_level == 0) { \
      std::string log_output; \
      llvm::raw_string_ostream rso(log_output); \
      block; \
      llvm::outs() << rso.str() << "\n"; \
    } \
  } while (0)

```

**宏定义**。

```cpp
PASS_LOG_DEBUG_BLOCK({
  rso << "Running Pass: " << passName << LOG_KV("time", time);
});
```

```
Running Pass: Conv2BN; time = 3.24 ms

```

####  5 `LAYER_GROUP_LOG_DEBUG_BLOCK`

```cpp
#define LAYER_GROUP_LOG_DEBUG_BLOCK(block) \
  do { \
    if (cur_log_level == 2 || cur_log_level == 0) { \
      ...
    } \
  } while (0)

```

#### 6  `SetLogFlag`

```cpp
inline void SetLogFlag(int32_t log_level) { cur_log_level = log_level; }
```

 **设置日志等级**。

#### 7  其他宏

```c++
#define LOG_KV(key, value) "; " << key << " = " << value
#define LOG_ACTION(action) "; action = " << action
#define LOG_LOC(file, line, func) "; file = " << file << "; line = " << line << "; func = " << func
```

```cpp
llvm::outs() << LOG_ACTION("run_pass") << LOG_KV("pass", "FoldConvBN") << "\n";
```

```
; action = run_pass; pass = FoldConvBN
```

#### 8  PROFILE_LOG 

```cpp
#define PROFILE_LOG(step, begin) \
  do { \
    DEBUG_WITH_TYPE("profile", { \
      auto current_time = std::chrono::high_resolution_clock::now(); \
      auto time_string = std::chrono::system_clock::to_time_t(current_time); \
      if (begin) { \
        llvm::dbgs() << LOG_ACTION("profile") << LOG_STEP(step) \
                     << LOG_KV("begin", std::ctime(&time_string)) << "\n"; \
      } else { \
        llvm::dbgs() << LOG_ACTION("profile") << LOG_STEP(step) \
                     << LOG_KV("end", std::ctime(&time_string)) << "\n"; \
      } \
    }); \
  } while (0)

```



```cpp
init_loglevel(1); 

PASS_LOG_DEBUG_BLOCK({
  rso << "Running optimization pass" << LOG_KV("pass_name", "ConvBNFold");
});

LAYER_GROUP_LOG_DEBUG_BLOCK({
  rso << "Layer group start" << LOG_KV("group_id", 3);
});
```

则只会输出，因为当前 log_level = 1，只显示 Pass 层级的日志。

```bash
Running optimization pass; pass_name = ConvBNFold
```

