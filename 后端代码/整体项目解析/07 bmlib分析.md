**Runtime**

AI 芯片的软件一般要分三层：

**lib 属于第二层（运行时层）**，它直接控制硬件，支持你编译好的 MLIR/TPU 指令正常执行。

**lib具体包含哪些内容**

驱动：TPU / VPU / JPU / VPP

内核模块（kernel module），主要功能是：

| 功能说明                                                     |
| ------------------------------------------------------------ |
| 负责 TPU 计算单元的控制、调度、指令下发、内存管理（分配 global mem / local mem） |
| 图像前处理（ISP）、视频输入模块                              |
| JPEG 编解码                                                  |
| 视频后处理（旋转/缩放/颜色空间转换）                         |

**TPU-MLIR 最终执行时必须通过 TPU 驱动才能运行真正的硬件指令**。

**运行时库（Runtime）**

运行时库是驱动的上层，主要帮助应用程序访问设备。

主要包含：

###  (1) **最底层 **

**这是最基础、最关键的库。**

功能包括：

- DMA（CPU <-> TPU）数据传输
- 分配 TPU 上的 global memory
- 把 TPU 指令下发到 TPU（load / run / wait）
- 查询设备信息
- 多芯片管理（node_idx）
- 多线程访问同步
- 错误处理

```
atomic_mm
get_local_mem(node_idx)
load cmdbuf
```

###  (2) **模型执行框架层）**

这一层把 TPU 的指令模型化为 program / sub-net / task。

- 解析 TPU-MLIR 生成的指令文件
- 管理模型的输入/输出 tensor
- pipeline & dynamic shape 调度
- 多芯片同步执行**你前端 MLIR 最终生成 b，runtime 会解释并执行它。**

###  (3) **图像/视频加速库**

为图像处理提供加速接口：

- resize
- crop
- color space convert
- normalize
- padding
- video decode/encode

TPU 计算前做图像前处理，所以 bmcv 用得很多。

**运行时库的开发环境**

这是为了让我能够调用 bmlib / bmcv / tpu runtime。

```
include/bmlib_runtime
include/bmcv_api_ext
include/bmrt
```

这些头文件定义了所有可用的API。

辅助工具

- 查看 bmodel 内容
- bm-smi：查看设备状态
- 优化、量化等工具
- 固件更新工具
- 设备查询工具

代码是否是在编译Runtime

经常用的

```
function build_bmlib_cmodel()
```

### 是在 **编译 bmlib 的 “cmodel 仿真版本”**。

- 不依赖真实硬件（TPU）
- 在 CPU 上模拟 TPU 指令执行
- 主要用于单元测试和后端开发调试

 TPU 必须渲染出 cmodel 来调试。

这里：

```
pushd $LIBSOPHON_TOP
mkdir -p build
cd build
cmake .. -DPLATFORM="cmodel"
make bmlib
```

= **进入项目的根目录，生成 cmodel 版本的.so**。