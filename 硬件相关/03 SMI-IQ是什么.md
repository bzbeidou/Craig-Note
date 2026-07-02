### 1 SMI-IQ

 **SMI 是什么**

- **SMI = Stream Multiprocessor Instruction Unit**
- 作用：
	- 接收 MID 解码后的微指令
	- 负责调度、发射到多个 SMC

**IQ 是什么**

- **IQ = Instruction Queue（指令队列）**
- 在这里是：
	- 基于 SRAM 的
	- 可写、可读
	- 带子队列（sub-queue）

**所以SMI-IQ就是SMI的一个组成部分**

SMI-IQ = SMI 内部的“微指令指令队列 + 发射控制逻辑”

```c++
宏指令（二进制）
  ↓
MID
  ↓  解码 / 展开
微指令（二进制）
  ↓
SMI
  ↓
SMI-IQ   ←←← 你现在看的模块
  ↓
SMC 内部模块（FPADD / MUL / UR / RtoVR 等）

```

SMI-IQ 是“微指令真正被执行前的最后一站”。



### 2 SMI-IQ是处理的微指令，非宏指令