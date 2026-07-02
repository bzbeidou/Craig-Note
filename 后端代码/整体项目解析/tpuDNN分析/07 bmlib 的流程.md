总体目标

**在 TPU 上执行一个卷积算子**。
 在上层（AI 编译器或者测试程序）调用 `tpudnnConvFloat`，然后通过一系列封装的接口，把卷积计算请求传递给 **TPU 硬件** 执行。

上层 API

```
return pimpl->launchKernel("sg_api_conv_float", &api, sizeof(api));
```

-  `TPUDNNImpl` 类型的对象
	1. 准备好输入、权重、偏置、输出、卷积核尺寸、步幅、padding 等
	2. 调用 `TPUDNNHost` 的 `launchKernel` 方法，把卷积请求发送出去把卷积操作从上层 API 转交给 Host 层来处理

Host 封装层：`TPUDNNHost::launchKernel`

```
tpudnnStatus_t launchKernel(const char *kernelName, void *arg, size_t argBytes) override {
    return mStream.launchKernel(kernelName, arg, argBytes);
}
```

- `TPUDNNHost` 是对具体设备（TPU）的高层封装
- 内部持有一个 **Stream 对象 `mStream`**
- Host 层负责做一些通用管理工作：
	- 创建/管理 Stream
	- 内存/缓存管理
	- Profiling（性能统计）
- 然后把 kernel 执行请求交给 Stream 层去具体执行

> **作用**：Host 层是“调度者”，把卷积计算请求发送给 Stream，屏蔽了底层复杂性。

Stream 层封装：`TPUStream::launchKernel`

```
tpudnnStatus_t launchKernel(const char *name, void *args, size_t arg_bytes) {
    return LauncherPolicy::launchKernel(name, args, arg_bytes);
}
```

- `TPUStream` 是对 **设备 Stream** 的抽象
- 内部依赖 **LauncherPolicy** 泛型（这里是 `BMLibLauncher`）
- Stream 层作用：
	- 把 Host 层传来的卷积任务映射到 **具体的硬件执行流**
	- 提供异步、多流、多核执行能力（可以支持多任务调度）

> **作用**：Stream 层是“流水线”，把计算请求发送给具体 Launcher。

Launcher 层封装

```
tpudnnStatus_t launchKernel(
    const char *name,
    void *args,
    size_t arg_bytes)
{
    auto func_id = tpu_kernel_get_function(mHandle.handle, mHandle.tpuModule, name);
    auto ret = tpu_kernel_launch(mHandle.handle, func_id, args, arg_bytes);
    if (BM_SUCCESS != ret)
    {
        LOG(FATAL) << "Failed to launch kernel \"" << name << "\" via bmlib";
    }
    return TPUDNN_STATUS_SUCCESS;
}
```

- 对 libsophon 提供的 **底层 runtime 接口** 的封装
	1. `tpu_kernel_get_function`
		- 获取函数 ID
	2. `tpu_kernel_launch`
		- 把 kernel ID 和参数发送到 TPU 上执行
- 这里完成了 **从 Host/Stream 到实际硬件执行** 的关键一步

动态库调用：

- `tpu_kernel_get_function` 和 `tpu_kernel_launch` 都在 **bmlib 动态库**里实现
- Host/Stream/Launcher 都不关心具体实现，只通过接口调用 bmlib
- bmlib 再通过 A53 Lite 或者 SMC/LSU 等机制，把计算真正下发到 TPU

> **作用**：动态库提供 **硬件驱动级别接口**，实现了任务的最终执行。

流程总结

整体流程可以这样理解：

```
tpudnnConvFloat()     
TPUDNNHost::launchKernel() 
TPUStream::launchKernel() 
BMLibLauncher::launchKernel() 
tpu_kernel_get_function() + tpu_kernel_launch()  
A53 Lite / SMC / TPU硬件执行卷积
```

> **核心思想**：
>
> - 分层封装
> - 每层只关注自己职责
> - 库负责把高层计算请求映射到硬件执行单元