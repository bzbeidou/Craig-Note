

#### 1、Module 

```cpp
module::init(mOp);
```

- 读取`CHIP`、`PLATFORM`、调试命令等

- 存到`module`命名空间下的全局变量，比如：

	- `module::chip` → 芯片类型

		`module::platform` → 平台类型

		`module::debug_cmd` → 调试命令

#### 2、Arch 硬件信息

```c++
backend::Arch::init(freq);
```

- 根据 `module::chip` 选择对应芯片实例,然后存到Arch的单例实例里

	```c++
	inst = &BM1684::instance();  // 每个芯片一个单例
	```

- 同时还设置环境变量 `CHIP`、`CORE_NUM` 方便其他模块读取



#### 3、日志和权重

- `module::init_loglevel(this->level)` → 存到全局日志状态
- `module::setWeightInMemFlag(weight_in_mem)` → 存到全局标志 `b_weight_in_mem`
