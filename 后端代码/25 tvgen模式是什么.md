代码体现

```
#ifdef SG_TV_GEN
#define FW_READ32_WAIT_GE(ADDR, WAIT_DATA, BITS, SHIFT) \
  sg_read32_wait_ge_tv(ADDR, WAIT_DATA, BITS, SHIFT, NODECHIP_REG);
```

- `SG_TV_GEN` 被定义 → 说明使用“TV 模拟”模式
- 宏调用 `sg_read32_wai_ge_tv` → 模拟硬件寄存器轮询的真实行为
- 宏里传了 `NODECHIP_REG` → 说明 TV 模拟有具体寄存器映射，保证行为和硬件一致
- TV 模拟和普通 CMODEL 的区别

| 特性      | 普通 CMODEL        | TV 模拟（SG_TV_GEN）                     |
| --------- | ------------------ | ---------------------------------------- |
| FIFO 行为 | 简单模拟           | 精细模拟 FIFO 逻辑，可能支持并发、多通道 |
| 指令执行  | CPU 内存里顺序执行 | 可能带时间步或周期精度，更接近硬件       |
| 寄存器    | 变量模拟寄存器     | 模拟具体寄存器布局，支持读写掩码位操作   |
| 测试向量  | 不一定生成         | 可以生成测试向量用于验证                 |

- **TV 模拟 = CPU 端的高级硬件行为模拟**
- 宏里调用 `sg_read32_wait_ge_tv` 就是执行这种模拟
- 非 TV 模拟（普通 CMODEL） → 宏可能为空操作，不做真实模拟
