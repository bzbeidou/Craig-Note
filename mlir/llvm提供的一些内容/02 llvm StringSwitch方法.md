#### 

- 输入一个变量str
- StringSwitch
- 如果是bm1684就返回这个枚举值Chip::BM1684

```c++
::std::optional<Chip> symbolizeChip(::llvm::StringRef str) {
  return ::llvm::StringSwitch<::std::optional<Chip>>(str)
      .Case("ALL", Chip::ALL)
      .Case("bm1684", Chip::BM1684)
      .Case("bm1684x", Chip::BM1684X)
      .Case("bm1688", Chip::BM1688)
      .Case("cv186x", Chip::CV186X)
      .Case("bm1690", Chip::BM1690)
      .Case("mars3", Chip::MARS3)
      .Case("sg2380", Chip::SG2380)
      .Case("sgtpuv8", Chip::SGTPUV8)
      .Case("cv180x", Chip::CV180x)
      .Case("cv181x", Chip::CV181x)
      .Case("cv182x", Chip::CV182x)
      .Case("cv183x", Chip::CV183x)
      .Default(::std::nullopt);
}
```

使用其Case方法进行对比

```c++
llvm::StringSwitch<ReturnType>(input_string)
    .Case("key1", value1)
    .Case("key2", value2)
    .Case("key3", value3)
    .Default(default_value);

```

`ReturnType`：返回类型模板参数

`.Case：定义匹配项；

`.Default：`

