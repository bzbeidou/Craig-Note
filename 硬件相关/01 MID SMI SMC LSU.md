#### AIACC芯片由MID SMC LSU四个部分组成



## 1 MID

宏指令译码单元（macro instruction decode unit）

将前端发送来的宏指令译码成微指令，微指令再传给SMI



- 宏指令可以由MID自行决定读取（**主动取，从片外DRAM或片上存储中取**）
- 或者由控制单元传过来（**主控处理器发送，类似CPU**）



**对AI编译器而言就是面对的宏指令**

因此这里AI编译器的结果是发送到DRAM或者给主控处理器、



宏指令通常对应：

- 一个算子（Conv / MatMul / Exp）
- 或一个算子阶段（load / compute / store）
- 或一个 tile/block 的计算



## 2 SMI

（Stream Multiprocessor Micro Instruction Unit）

流多处理器的微指令相关单元，**芯片有很多个SMC，可以同时处理多个操作，SMI可以接收这些微指令，然后将其自动的分配给SMC，不需要自己控制**

- 接收MID发送的微指令
- 向多个流多处理器SM发射微指令



**编译器在这方面的相关工作，很少**

**并行性是“隐式的”**

- 你不需要手动控制“发给第几个 SMC”
- 你只需要：
	- 指定并行规模
	- 指定数据布局 / tile 方式

编译器需要做的是：

- **暴露足够的数据并行性**
- 避免串行依赖



## 3 SMC 就是执行单元

流多处理单元的计算单元（Stream Multiprocessor Compute Unit）

- 真正进行 **数值计算**
- 每个 SMC 内部通常包含：
	- 向量 ALU
	- 矩阵 / Tensor 单元
	- SFU（exp、div、sqrt 等）

**对 AI 编译器意味着什么**

**算子最终一定要映射到 SMC 能力范围内**

- 数据类型是否支持
- 是否支持该算子（或需要拆分）



## 4 LSU 存取单元 （Load Store Unit）

#### 硬件在做什么

- **显式搬运数据**
	- 外部 → AIACC 内部（SRAM / local memory）
	- AIACC 内部 → 外部
- 不像 CPU/GPU cache 那样透明

这是典型的 **显式数据流架构**

对 AI 编译器意味着什么（非常非常重要）

**数据搬运是编译器的责任**

- 什么时候 load
- load 多大
- 放在哪里
- 什么时候 store

编译器必须做：

- memory planning
- buffer 生命周期管理
- tile + DMA 调度

这意味着：

- MLIR 中一定会有
	- explicit buffer
	- explicit copy / dma op