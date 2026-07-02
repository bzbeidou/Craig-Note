```c++
  if (param.is_L_const) param.L_addr = L_val.u32val;
  if (param.opd2_is_const) param.opd2_addr = opd2_val.u32val;
  CMD_ID_NODE pid_node;
  init_cmd_id_node(&pid_node);
  gen_cmd(&param, &pid_node);
  poll_all_engine_done(&pid_node); // 等待硬件执行完指令
  gather_data_from_lmem(lmem, &Y_info, Y_got, false);
```

**真实硬件（TPU芯片）**

会变成 **等待硬件执行完成的关键代码**

- `FW_READ32_WAIT_GE`
- 等待 BD/GDMA 引擎执行完当前 CMD ID
- 防止你提前读取结果导致错误数据

**模式 2：CModel + 多线程引擎）**

会执行：

```
while(!all_engine_done)
    all_engine_done = check_all_engine_done(...);
```

- 执行在不同线程
-  需要等待所有线程完成

也就是软件层面的“等待执行完成”。