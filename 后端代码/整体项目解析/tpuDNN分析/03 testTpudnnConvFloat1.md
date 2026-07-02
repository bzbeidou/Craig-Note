

## TestOption option

```c++
  TestOption option;
  option.add_seed(&seed)
      .add_loop(&loop)
      .add_enum(&scale, "scale", 's', "Test Size",
                {{0, "random"},
                 {1, "tiny"},
                 {2, "small"},
                 {3, "medium"},
                 {4, "large"}})
      .add_NCHW(&param.n, &param.ic, &param.ih, &param.iw, "input")
      .add_int(&param.oc, "out_channel", 'o', "output channel")
      .add_int(&param.multicore, "multicore", 'm', "using multi core")
      .add_dtype(&type, "dtype", 't', "",
                 {SG_DTYPE_FP32, SG_DTYPE_FP16, SG_DTYPE_BFP16,
                  SG_DTYPE_FP8E5M2, SG_DTYPE_FP8E4M3})
      .parse(argc, argv);

```

`TestOption option;`

- 创建一个命令行参数解析器对象 `option`。

	

 `.add_seed(&seed)`

- 给命令行添加一个参数 `-S` 或 `--seed` 用来指定随机数种子。
- 绑定到变量 `seed` 上。
- 例子：

```
./testTpudnnConvFloat -S 12345
```

执行后，`seed` 会被设置为 `12345`。
 如果命令行没传，就会用 `time(0)` 生成默认值。

`.add_loop(&loop)`

- 添加一个参数控制
- 绑定到 `loop` 变量。
- 例子：

```
./testTpudnnConvFloat -L 5
```

`.add_enum(&scale, "scale", 's', "Test Size", {...})`

- 添加枚举类型参数
- 对应变量 `scale`，可选值如下：

| 值   | 描述   |
| ---- | ------ |
| 0    | random |
| 1    | tiny   |
| 2    | small  |
| 3    | medium |
| 4    | large  |

- 例子：

```
./testTpudnnConvFloat -s 2
```

会将 `scale` 设置为 `2`

 `.add_NCHW(&param.n, &param.ic, &param.ih, &param.iw, "input")`

- 添加 **输入 tensor 的形状参数**：
	- `n` = batch size
	- `ic` = input channels
	- `ih` = height
	- `iw` = width
- 绑定到 `conv_float_param_t` 的对应字段。
- 例子：

```
./testTpudnnConvFloat --input 1 3 224 224
```

会把 `param.n=1, param.ic=3, param.ih=224, param.iw=224`。

`.add_int(&param.oc, "out_channel", 'o', "output channel")`

- 添加一个整型参数 `-o` 或 `--out_channel`，对应输出通道数。
- 例子：

```
./testTpudnnConvFloat -o 64
```

会将 `param.oc = 64`。

`.add_int(&param.multicore, "multicore", 'm', "using multi core")`

- 添加整型参数控制是否使用多核 TPU。
- 例子：

```
./testTpudnnConvFloat -m 1
```

会将 `param.multicore = 1`（开启多核）。

`.add_dtype(&type, "dtype", 't', "", {...})`

- 添加数据类型参数 `-t` 或 `--dtype`。
- 可选值是 SG_DTYPE_FP32 / FP16 / BFP16 / FP8E5M2 / FP8E4M3。
- 例子：

```
./testTpudnnConvFloat -t 0
```

会将 `type = SG_DTYPE_FP32`。

`.parse(argc, argv)`

- 解析命令行参数，把用户输入填充到上面绑定的变量里。
- 同时，如果用户输入 `-h` 或参数错误，会打印帮助信息。