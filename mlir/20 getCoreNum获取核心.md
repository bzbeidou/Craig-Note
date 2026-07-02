```c++
setenv("CORE_NUM", std::to_string(module::getCoreNum()).c_str(), 1);


int64_t getCoreNum() {
  if (auto cores = m->getAttrOfType<IntegerAttr>(Attr::CORES))
    return cores.getInt();
  return 1;
}

	static constexpr llvm::StringRef CHIP = "module.chip";
  	static constexpr llvm::StringRef CORES = "module.cores";
```

#### 设定的使用核心数量

```mlir
module {"module.chip" = "bm1684", "module.cores" = 2 } {
}
```

- 设定了使用的芯片是`bm1684`，使用的核心数是`2`个
- 直接存到环境变量里使用

