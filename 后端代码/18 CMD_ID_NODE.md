#### 首先是一个结构体

```c++
typedef struct cmd_id_node {
  // ATTENTION:
  // DO NOT CHANGE THE ORDER OF THE FOLLOWING MEMBERS
  // IF YOU WANT TO ADD NEW MEMBERS, PLEASE ADD THEM AT THE END
  unsigned int bd_cmd_id;
  unsigned int gdma_cmd_id;
  unsigned int hau_cmd_id;
  bool in_parallel_state;
#if defined(SG_STAS_GEN) || defined(SG_TV_GEN)
  long long cycle_count;
  long long cur_op_cycle;
#endif
#ifdef SG_STAS_GEN
  char cmd_name[16];
  char name_prefix[64];
  gdma_cmd_node_info_t gdma_cmd_info;
  INST_PROFILE inst_profile;
#endif
  unsigned int sdma_cmd_id;
  unsigned int cdma_cmd_id[CONFIG_MAX_CDMA_NUM];
  unsigned int vsdma_cmd_id[CONFIG_MAX_TPU_CORE_NUM];
  unsigned int slave_bd_cmd_id;
  unsigned int slave_gdma_cmd_id;
  bool in_sync_state;
  int hybrid_engine_bitmap;
} CMD_ID_NODE;
```

### **一、CMD_ID_NODE 是干什么的？**

**它是当前 OP 对应的所有类型指令（BD/GDMA/HAU/SDMA/CDMA…）生成的 ID 的集合 + 状态记录。**

也就是说：一个算子（比如 MatMul/LUT/Conv）在模拟器里会生成多种类型的指令，CMD_ID_NODE 就是用来记录它们每种指令的编号，方便后续执行、同步、统计性能等。

**理解：一个算子会有一个对应的结构体，这个结构体里记录指令的情况**

### 二、逐字段解释

代码里写了“不能改变顺序”，说明解析器依赖具体布局。

------

## 1. **unsigned int bd_cmd_id;**

BD = **BDMA（block dma）**

意思是：

- 如果这个 OP 产生 BDMA 指令，给每条 BDMA 指令分配一个 ID
- bd_cmd_id 存的是当前 op 的“最后一条”（或最近）BDMA 指令的 ID

用途：

- 之后别人要同步 BDMA 指令，就可以用这个 ID
- 用于 trace dump 里查是哪条指令

------

## 2. **unsigned int gdma_cmd_id;**

GDMA = **Global DMA（搬运 global → local 或反向）**

大部分 OP 在开始阶段都会生成 GDMA 指令，所以这是最常用的 ID。

它的作用：

- 记录当前 OP 图片涉及到的所有 GDMA 指令的 ID
- 后续执行顺序、依赖 sync 时需要用到

------

## 3. **unsigned int hau_cmd_id;**

HAU = **Hardware Acceleration Unit（AI 运算核心，如 MatMul/Conv）**

也就是 **真正执行算子的 TPU 核心指令的 ID**。

例如：

- MatMul → HAU 指令
- Convolution → HAU 指令

## 举例解释调用实际指令ID的生成

MatMul_LUT 的实际使用流程

例如执行 MatMul_LUT：

1. 调用 api_matmul_lut_global()
2. GDMA：搬输入 → local
	 → 产生 gdma_cmd_id = 20，**很简单这里就是对结构体的一个成员变量赋值而已**
3. BDMA：局部排布 → local
	 → 产生 bd_cmd_id = 7
4. HAU：执行 matmul
	 → 产生 hau_cmd_id = 33
5. SDMA：写回 global
	 → sdma_cmd_id = 5


