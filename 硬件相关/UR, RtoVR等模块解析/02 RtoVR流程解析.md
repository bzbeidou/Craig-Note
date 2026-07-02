## 工作流程（结合数据转发场景）

以 “UR 数据→RtoVR→FPADD” 为例，看两个寄存器的协同：

1. UR 通过 “UR-RDATA-0”（RtoVR 输入信号 IN30）输出 128bit 数据；
2. 数据先存入 “SMC [*]/IN30-PIPE” 寄存器，缓冲 1 个时钟周期（解决 UR 输出与 RtoVR 转发的时序差）；
3. RtoVR 按微指令，将 IN30-PIPE 中的数据转发到 “SMC [*]/OUT5-PIPE”（OUT5 对应 FPADD 的输入接口 DVR-FPADD-S0）；
4. OUT5-PIPE 中的数据再缓冲 1 个时钟周期，确保 FPADD 接口准备就绪后，再输出到 DVR-FPADD-S0。