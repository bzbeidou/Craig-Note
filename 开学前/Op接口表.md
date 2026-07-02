| 方法                               | 作用                                    |
| ---------------------------------- | --------------------------------------- |
| `getFLOPs()`                       | 返回该算子的浮点运算量                  |
| `init(InferenceParameter &p)`      | 推理前的初始化                          |
| `deinit(InferenceParameter &p)`    | 推理后的清理                            |
| `inference(InferenceParameter &p)` | 核心计算逻辑：遍历输入                  |
| `shape_inference()`                | 根据输入 shape 和算子属性推导输出 shape |

| 方法                      | 作用                                             |
| ------------------------- | ------------------------------------------------ |
| `init` / `deinit`         | 和 Top 类似                                      |
| `inference`               | 核心计算                                         |
| `BackwardH` / `BackwardW` | 计算卷积/矩阵乘分块时 H/W 方向的边界 tile offset |
| `LocalGenSupport`         | 是否可以在本地核心生成/运行                      |
| `DumpQuantAgnosticAttrs`  | 输出与量化无关的属性                             |
| `assign_sec_info`         | 分配二级信息                                     |
| `assign_fw_param`         | 前向执行分配参数                                 |
| `support_multi_core`      | 检查是否支持多核并行                             |