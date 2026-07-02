```c++
#define FW_READ32_WAIT_GE(ADDR, WAIT_DATA, BITS, SHIFT)
```

```
FW_READ32_WAIT_GE(some_addr, 5, 4, 0);
```

- 预处理器会把它替换成**空白**：

```
;  // nothing here
```

- 程序执行时，相当于这一行不存在，不会做任何事情

在普通 CMODEL 模式（非 TV 模拟）下：

```
#define FW_READ32_WAIT_GE(ADDR, WAIT_DATA, BITS, SHIFT)
```

- 宏展开为空操作

这块内存确实是模拟 FIFO，但普通 CMODEL 并没有**真正实现 FIFO 的逻辑控制**：

- 没有读/写指针
- 没有容量限制
- 写满了也不会阻塞

2 为什么可以这么做

**快速模拟 CPU 下的指令生成逻辑**，不关注硬件 FIFO 的行为：

1. **测试主要是验证算法或指令生成顺序**
2. **性能和阻塞行为不是重点**
3.  和 TV 模拟的区别

- **TV 模拟/真实硬件**：
	- FIFO 
	- CPU/模拟器要轮询状态寄存器
	- `FW_READ32_WAIT_GE` 会实际检查 FIFO 是否有空位
- **普通 CMODEL**：
	- FIFO 内存
	- 写入直接覆盖
