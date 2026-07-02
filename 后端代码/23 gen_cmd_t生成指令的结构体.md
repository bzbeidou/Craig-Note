```c++
typedef struct GEN_CMD {
    cmd_t cmd;
    P_COMMAND (*buf)(void);
    void (*id_proc)(unsigned int *, CMD_ID_NODE *);
    void (*set_id)(cmd_t *, const unsigned int *);
    void (*common)(cmd_t *);
    void (*start)(cmd_t *);
} gen_cmd_t;
```

```c++
typedef struct CMD {
    P_COMMAND buf;
    int len;
    void (*write)(P_COMMAND, unsigned int, unsigned int);
    unsigned int (*read)(P_COMMAND, unsigned int);
} cmd_t;
```

```c++
typedef void* P_COMMAND;
```

## 先看cmd_t

```
typedef struct CMD {
    P_COMMAND buf; 
    int len;
    void (*write)(P_COMMAND, unsigned int, unsigned int);
    unsigned int (*read)(P_COMMAND, unsigned int);
} cmd_t;
```

### 1 `P_COMMAND buf;`

- **类型**：`P_COMMAND`，实际上是 `typedef void* P_COMMAND;`

------

### 2 `int len;`

- **作用**：
  - 表示命令缓存区的长度
  - 可能单位是：
  	- BD 条数（每条 BD 由 high/low 两个 u64 组成）
  	- 或者数组元素数（u64 数组长度）

------

### 3 `void (*write)(P_COMMAND, unsigned int, unsigned int);`

- **功能**：
  - 把高低 64-bit 指令写入命令缓存

------

### 4 `unsigned int (*read)(P_COMMAND, unsigned int);`

- **作用**：
  - 从命令缓存读取数据
  - 返回值：
  	- 索引位置的值
- **功能**：
  - 模拟或者实际硬件读取命令或状态



