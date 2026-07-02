**功能：都是将控制字段（就是 c++的结构体对象内容）打包成微指令，然后送到硬件去执行的**



结合代码来看很好理解

就两步

- `pack_cru`打包对应操作的指令码（对应当前操作的内容）
- `make_micro_inst`生成一个 `MicroInstruction` 对象（这是IQ指令的统一规范）

## 1. **IntAddIq**

- **作用**：生成整数加法指令（IntAdd）的微指令
- **核心流程**：
	1. 拿到一个 `IntAddCru` 结构（里面有 S0/S1/S2 精度、符号、update 标志等）
	2. 调用 `pack_cru_intadd` 按位打包成 12bit 指令码
	3. 用 `make_micro_inst` 生成一个 `MicroInstruction` 对象
	4. push 到 IQ 或 sink
- **关键点**：
	- 每条 IntAdd 微指令只占 1 个 `MicroInstruction`
	- IQ 类负责把 C++ 结构映射到硬件能识别的 bit 流

------

## 2. **RtoVrIq**

- **作用**：把寄存器到向量寄存器（RtoVR）的控制字段打包
- **核心流程**：
	1. 拿到 `RtoVrCru`（里面包含 40 个输出控制单元，每个 6bit）
	2. `pack_cru_rtovr_words` → 生成 8 个 32bit word（总 240bit，硬件宽度对齐 256bit）
	3. `make_micro_insts` → 生成 8 条 `MicroInstruction`
- **关键点**：
	- 一次操作会生成多条微指令（8条）
	- 比 IntAdd 更复杂，因为寄存器映射和宽度更大

------

## 3. **UrIq**

- **作用**：UR（User Register）读写指令生成器
- **核心流程**：
	1. 拿到 `UrCru`（包括 4 个写地址、4 个读地址、re/we/wbe/vld 等）
	2. `pack_cru_ur_words` → 打包成 6 个 32bit word（总 192bit）
	3. `make_micro_insts` → 生成 6 条 `MicroInstruction`
- **关键点**：
	- 专门针对 UR 读写
	- 一次操作生成多条微指令（6条）
	- 可用作 load/store 数据到 UR