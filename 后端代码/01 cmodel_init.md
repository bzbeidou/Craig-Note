处理非规格化数

```
#ifdef DISABLE_DENORM  
  set_denorm();
#endif
```

避免 CPU 处理 denormal 浮点数时降低 100x 速度。置 MXCSR 寄存器，开启：FTZ DAZ多线程引擎

```
#ifdef USING_MULTI_THREAD_ENGINE
  cmodel_multi_thread_cxt_init(node_idx);
#endif
```

多 NPU 并行计算模拟。
模拟多个线程并行跑指令

- 创建线程池

- 创建执行队列

- 分配并初始化 per-thread context

	芯片是否已初始化过

```
if (cmodel_initialized[node_idx]) {
  printf("Initialized already\n");
  return CMODEL_SUCCESS;
}
cmodel_initialized[node_idx] = 1;
```

`cmodel_initialized` 

```
static int cmodel_initialized[MAX_NODECHIP_NUM] = {0};
```

node_idx 表示一块独立 TPU。

分配 GM

```
alloc_continuous_mem(&continuous_mem[node_idx][MEM_TYPE_GLOBAL],
                     GLOBAL_MEM_START_ADDR, global_mem_size, MEM_TYPE_GLOBAL);

global_mem_arr[node_idx] = continuous_mem[node_idx][MEM_TYPE_GLOBAL].raw_ptr;
```

- 用于大张量
- 作用类似“全局内存”

`continuous_mem[][]` 是一个数组：

```
continuous_mem[node][mem_type]
```

每个 mem_type 有一个 `CONTINUOUS_MEM` 来表示：

- 地址
- 可 free
- raw_ptr

实际 raw_ptr 存到：

```
global_mem_arr[node_idx]分配 LS
```

```
alloc_continuous_mem(&continuous_mem[node_idx][MEM_TYPE_L2],
                     L2_SRAM_START_ADDR, L2_SRAM_SIZE, MEM_TYPE_L2);
l2_sram_arr[node_idx] = continuous_mem[node_idx][MEM_TYPE_L2].raw_ptr;
```

- Weight cache
- Input cache
- DMA 中转

分配

```
alloc_continuous_mem(&continuous_mem[node_idx][MEM_TYPE_STATIC],
                     STATIC_MEM_START_ADDR, STATIC_MEM_SIZE, MEM_TYPE_STATIC);
static_mem_arr[node_idx] = continuous_mem[node_idx][MEM_TYPE_STATIC].raw_ptr;
```

LM

```
alloc_continuous_mem(&continuous_mem[node_idx][MEM_TYPE_LOCAL],
                     LOCAL_MEM_START_ADDR, LOCAL_MEM_SIZE * NPU_NUM, MEM_TYPE_LOCAL);
```

 独享的高速内存

```
32 个 NPU × 128 
```

切 NPU 

```
make_local_mem_from_continuous(&local_mem_arr[node_idx], NPU_NUM, LOCAL_MEM_SIZE,
                               EU_NUM, &continuous_mem[node_idx][MEM_TYPE_LOCAL]);
```

```
local_mem_arr[node_idx].mem_arr[0] → NPU0 的 local mem
local_mem_arr[node_idx].mem_arr[1] → NPU1 的 local mem
...
local_mem_arr[node_idx].mem_arr[31] → NPU31 的 local mem
```

填充 lm

```
fill_local_mem(node_idx);
```

分配数组区域

```
arrange_reg_arr[node_idx] = new u32[get_eu_num(INT32) * NPU_NUM];
```

- get_eu_num(INT32) = 每个 NPU 的

TV Generator 

```
#ifdef SG_TV_GEN
  memset(global_mem_arr[node_idx], 0x00, global_mem_size);
  int *initial = (int *)global_mem_arr[node_idx];
  ...
  if (sg_tv_gen_init(node_idx) == CMODEL_FAIL) {
    return CMODEL_FAIL;
  }
#endif
```

SG 初始化

```
#ifdef SG_STAS_GEN
  sg_stas_gen_init(node_idx);
#endif
```

STAS = profile 或统计信息生成器

