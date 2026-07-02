### 直接调用`rebuild_test`

### （1）firmware

- 构建firmware

- 构建test

### （2）test

默认是sgdnn模式，

- sgdnn下开启的选项有：

`BUILD_CONFIG = -DUSING_CMODEL=ON -DUSING_SGDNN_TEST=ON -DFAST_GEN_CMD=ON -DUSING_TPUDNN=ON `

- 输出在

`build_test文件夹`

所以一个命令版本就是

```bash
cmake -S . -B build_test -DUSING_CMODEL=ON -DUSING_SGDNN_TEST=ON -DFAST_GEN_CMD=ON -DUSING_TPUDNN=ON ${EXTRA_CONFIG}

cmake --build build_test -- -j
```

### （3） 生成了哪些代码

包括三个文件夹`tpuDNN， runtime, sccl`