#### 初始化指令代码

每次生成指令的时候就会调用这个初始化代码，为什么？

```c++
void init_cmd_id_node(CMD_ID_NODE *p_id_node) {
  p_id_node->bd_cmd_id   = 0;
  p_id_node->gdma_cmd_id = 0;
#if defined(SG_STAS_GEN) || defined(SG_TV_GEN)
  p_id_node->cycle_count = 0;
  p_id_node->cur_op_cycle = 0;
#endif
#if defined(USING_MULTI_THREAD_ENGINE) && defined(USING_CMODEL)
    reset_all_engine_cmd_id(get_cur_nodechip_idx());
#endif
}
```

进行初始化：

```
p_id_node->bd_cmd_id   = 0;
p_id_node->gdma_cmd_id = 0;
p_id_node->hau_cmd_id  = 0;  ← 有些平台有，有些没有
...
```

也就是说：**把所有 BD、GDMA、HAU、SDMA… 的指令计数器清零**

为什么要清零？

因为每个 Layer（如 matmul、conv）执行时，会生成一堆硬件指令：

- GDMA 指令
- BD 指令
- HAU 
- SDMA/CDMA

这个结构体保存了这些 ID。

**问题：这里指令ID不像是ID，反而更像是生成指令的次数**