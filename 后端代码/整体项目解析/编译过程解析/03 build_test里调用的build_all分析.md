```bash
  build_all "${BUILD_CONFIG}" "${TEST_DIR}" || return $?
```

- build_test
	- uild_all
		-  BUILD_CONFIG  TEST_DIR
		- BUILD_CONFIG  如下
			- `"-DUSING_CMODEL=ON -DUSING_${UPPER_TYPE}_TEST=ON $FAST_GEN_CMD_FLAG"` 这个UPPER_TYPE默认是sgdnn，后面的变量是在具体的某个硬件里定义的`FAST_GEN_CMD_FLAG=" -DFAST_GEN_CMD=ON"`
			- 条件选择，不为cmodel情况下直接开启多线程`BUILD_CONFIG+=$CMODEL_RUNTIME_FLAG`  `export CMODEL_RUNTIME_FLAG=" -DUSING_MULTI_THREAD_ENGINE=ON"`
			- 条件选择，如果是sgdnn情况添加一个`BUILD_CONFIG+=" -DUSING_TPUDNN=ON" `
			- 可选`BUILD_CONFIG+=" -DUSING_SGDNN_TEST=ON"`

