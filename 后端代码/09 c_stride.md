## 1

这部分代码就是为了获取一个channel占多少个空间

- 对于左矩阵来说
- 所以这里输入是一行的内容，行数是1，然后宽度就是列数
- 这里返回一个channel的一行有多少个数据
- 当然是针对具体数据类型`LR_prec`的
- EU处理数量

```c++
int c_stride = get_local_cstride(1, param.L_tensor_W, true, param.LR_prec);

INLINE static int get_local_cstride(int h, int w, bool align, PREC precision) {
  return align ? ALIGN(h * w, get_eu_num(precision)) : (h * w);
}

#define ALIGN(x, a) (((x) + (a) - 1) / (a) * (a))
```



## 2

- 放了几个channel

- 这几个channel再乘
- 然后再乘每个数据的byte数
- 当前芯片左矩阵
- 在下一步就是设置右矩阵的信息

```c++
start_local_offset += get_local_nstride(c_stride, param.L_tensor_C, param.L_addr)* (param.is_L_trans ? param.L_col_num : param.L_row_num)* get_bytesize(param.LR_prec); 


INLINE static int get_local_nstride(int c_stride, int c, u32 local_addr) {
  local_addr = local_addr | LOCAL_MEM_START_ADDR; // 获取绝对地址
  int npu_idx = (local_addr - LOCAL_MEM_START_ADDR) >> LOCAL_MEM_ADDRWIDTH; // 获取当前地址对应的哪一块TPU空间，这里和前面的L_start_npu是数值完全相同，但用途不同
  return (ceiling_func_shift(c + npu_idx, NPU_SHIFT) * c_stride); 
}
```

## 3

这一步就是再对start_local_offset这个数据进行一个64 位的对齐，但是不对齐应该也没问题

这个`start_local_offset`后面就是可以作为右矩阵存储的起点来用了

```c++
start_local_offset = ALIGN(start_local_offset, 64); 
```

