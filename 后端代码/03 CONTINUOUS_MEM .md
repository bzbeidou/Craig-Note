- 分配空间模拟TPU的某个类型的空间
- 开辟方式使用的创建类型数组
- 模拟的内存

```c++
typedef struct {
  char* raw_ptr;
  u64 start_addr;
  u64 size;
  int type;
  LOCAL_MEM *p_local;
} CONTINUOUS_MEM;

void alloc_continuous_mem(CONTINUOUS_MEM *mem, u64 start, u64 size, int type) {
  ASSERT(mem != NULL);
  mem->raw_ptr = new char[size];
  mem->type = type;
  mem->start_addr = start;
  mem->size = size;
  mem->p_local = NULL;
  ASSERT_INFO(mem->raw_ptr != NULL, "alloc_continues_mem failed");
  return;
}

```



```
CONTINUOUS_MEM mem;
alloc_continuous_mem(&mem, 0x1000, 16, 1);
```

```
CONTINUOUS_MEM mem
│ raw_ptr ──────────▶ [ byte0 ]
│ start_addr
│ size      
│ type      

```

 `p_local`分配

```
mem.p_local ──▶ LM
                 ┌─────────────┐
                 │ mer[0] ─▶ [..] │
                 │ mer[1] ─▶ [..] │
                 │ ...            │
                 └─────────────┘
```

