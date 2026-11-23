## tpuDNN和test之间的区别

**`bm1684x/test` 是“指令级”底层测试——直接调用 `atomic_xxx_gen_cmd`、直接往 LMEM 里手工摆数据、只链接 `firmware_base + cmodel`；**

**tpuDNN 是“全栈”测试——从 host API 一路穿过 bmlib、固件路由、nodechip 切分，最后才到达同一个 atomic。** 前者测“一条指令对不对”，后者测“整条链通不通”。



## 放进全景：这套仓库的四层测试体系

```
┌────────────────────────────────────────────────────────────┐
│ tpuDNN/tests + tests_py        【全栈】host API → 一路到底   │
│   （替代已废弃的 test/test_api_sgdnn）                        │
├────────────────────────────────────────────────────────────┤
│ test/test_layer                【中层】nodechip / tpu_kernel  │
│   （还能用底层代码，但头文件顺序有讲究）                        │
├────────────────────────────────────────────────────────────┤
│ bm1684x/test（各芯片一套）      【底层】atomic + cmodel 直测   │
├────────────────────────────────────────────────────────────┤
│ bm1684x/test_pld               【硬件】PLD/仿真器上跑         │
└────────────────────────────────────────────────────────────┘
```

越往上越接近真实使用方式，越往下问题定位越精确。

------

## 为什么两层都要存在？——调试的二分法

这是最实用的价值：

```
tpuDNN 的卷积 case 挂了
   │  可能是：tpuDNN 封装 / bmlib / 消息路由 / nodechip 切分 / tpu_kernel / atomic / cmodel
   ▼
跑 bm1684x/test 的 test_conv_cmodel
   ├── 也挂了 → 问题在 atomic 或 cmodel（范围瞬间缩小到 1/6）
   └── 没挂   → 问题在中间层（nodechain 切分 / 路由 / bmlib），atomic 是好的
```

反过来，**有些东西只有底层测试能覆盖**：`test_conv_cmodel.cpp` 能直接传 `w_is_const=1`、`pad_ins_is_const=0` 这种组合去打 atomic 的所有分支——这些开关在 tpuDNN 的 `tpudnnConvFloat` 里根本不暴露，全栈测试永远测不到它们。指令集文档（我们逆向的那份）里的每一个字段组合，对应的就是这个目录里的测试用例。

------

## 同一个卷积，两种写法对比

```
// ── bm1684x/test：直接对指令编程 ───────────────────
tensor_info_generate(...);                     // 自己算 NPU 布局
scatter_data_to_lmem(get_local_mem(0), ...);   // 自己把数据摆进 LMEM
atomic_conv_gen_cmd(iaddr, waddr, ..., p_is_const ? pconst : paddr, ...);
// 然后从 LMEM 把结果取出来和 CPU 参考值对拍

// ── tpuDNN：像用户一样调用 ────────────────────────
tpudnnMemcpyHostToDevice(handle, input_tpu, input, size);   // 走 GDMA
tpudnnConvFloat(handle, input_tpu, weight_32IC, ...);        // 走完整条链
tpudnnSync(handle);
tpudnnMemcpyDeviceToHost(handle, output, output_tpu, size);
```

有趣的是两者**共享同一个 CPU 参考实现**（都 include `conv_util.h` 的 `conv_native_core`）——区别只在于“通往 TPU 的路走了多长”。