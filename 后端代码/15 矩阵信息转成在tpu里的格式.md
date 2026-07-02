- 矩阵信息（包含左右矩阵）
- TENSOR_INFO
	- 第一行，然后第二行，一直存下去
- localmem并不是这样，并不是完整的一个矩阵
- 所以在得到一个cpu上的矩阵信息之后需要将其进行分析从而得到一个TENSOR_INFO的信息，其中包含了如何切分的信息
- 疑问，这里TENSOR_INFO里的nchw代表的是图像里的nchw的含义吗？**并不是**
- **下面具体讲一下**

```c++
typedef struct
{
  int L_row_num;
  int L_col_num;
  int R_col_num;
  int L_tensor_C;
  int L_tensor_W;
  int R_tensor_C;
  int R_tensor_W;
  int is_L_trans;
  int is_L_const;
  int add_result;
  int AddResLShiftBits;
  int rshiftbits;
  PREC LR_prec;
  PREC Y_prec;
  PREC opd2_prec;
  int L_sign;
  int R_sign;
  int opd2_sign;
  int opd2_enable;
  int opd2_is_const;
  DataUnion opd2_val;
  ROUND_MODE round_mode;
  MM_OP mm_op;
  u32 L_addr;
  u32 R_addr;
  u32 opd2_addr;
  u32 Y_addr;
} MM_PARAM; 
```

```c++
typedef struct tensor_info {
  u32 n, c, h, w;
  union {
    u32 str_data[4];
    struct {
      u32 n_stride;
      u32 c_stride;
      u32 h_stride;
      u32 w_stride;
    };
  };
  u32 address;
  PREC precision;
  u32 neuron_matrix;      // 0: neuron, 1: matrix
  u32 matrix_col_margin;  // the margin is not 0, when column_num%w_param!=0
} TENSOR_INFO;
```

##  切分

假设tile = 4； 按列切分那么就需要用到3个tile

| tile 编号 | 覆盖列 |
| --------- | ------ |
| tile0     | 0~3    |
| tile1     | 4~7    |
| tile2     | 8~9    |

 **scatter_data 的存储 layout 决定：**

**每个 tile 的所有行按行写入同一个 NPU**

tile0 的数据（每行前 4 个元素）, 很明显这里是第一个tile的内容：

```
NPU0:
[ a00 a01 a02 a03 a10 a11 a12 a13 ... a40 a41 a42 a43 ]
```

5行 × 4 列 = 20 个元素
 c_stride = 20

- Tile1 在 NPU1（存第二个tile，和上面是重复的）

tile1

```
r0: a04 a05 a06 a07
r1: a14 ...
...
r4: a44 a45 a46 a47
```

```
NPU1:
[ a04 a05 a06 a07 a14 a15 a16 a17 ... a44 a45 a46 a47 ]
```

tile2（单行只有 2 列：a08, a09，后面会 pad 2 个位置）：

```
r0: a08 a09 xx xx   (xx = padding，不写入)
r1: a18 a19 xx xx
...
r4: a48 a49 xx xx
```

```
NPU2:
[ a08 a09 a18 a19 a28 a29 a38 a39 a48 a49 ]
```

```
NPU0 (tile0):
[ a00 a01 a02 a03 | a10 ... | a40 a41 a42 a43 ]

NPU1 (tile1):
[ a04 a05 a06 a07 | a14 ... | a44 a45 a46 a47 ]

NPU2 (tile2):
[ a08 a09         | a18 a19 | ... | a48 a49 ]
```

## 结合代码

**（1）5×10 矩阵为例**

- ```c++
	L_row_num = 5;
	L_col_num = 10;
	L_addr = 某地址；
	```

**（2）有了矩阵参数信息**

- 先针对参数信息开辟空间mollo）
- 再往里随机填数据

```c++

  void *L_ptr = NULL, *L_trans = NULL;  
  if (!param.is_L_const) {
    L_ptr = malloc(param.L_row_num * param.L_col_num * get_bytesize(param.LR_prec)); 
    mm_gen_data(L_ptr, param.L_row_num * param.L_col_num, param.LR_prec); 
    L_trans = malloc(param.L_row_num * param.L_col_num * get_bytesize(param.LR_prec));
    mm_trans(L_ptr, L_trans, param.L_row_num, param.L_col_num, param.LR_prec); 
  }
```

**（3）然后locmem**

- 到locmem里就要切分
- 因此要注意索引之间的对应关系才能搬