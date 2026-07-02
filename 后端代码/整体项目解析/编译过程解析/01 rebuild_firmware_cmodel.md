#### 顺序

- build_test
- rebuild_firmware_cmodel
	- 参数`${FIRMWARE_BUILD_CONFIG}`,其内容列出来
		- `FIRMWARE_BUILD_CONFIG="-DUSING_${UPPER_TYPE}_TEST=ON"`，必有，这里`UPPER_TYPE`默认是SGDNN
		- `FIRMWARE_BUILD_CONFIG+=" -DCORE_ID=$CORE_ID"`条件有，在`"$CORE_ID"`的条件下有
		- `FIRMWARE_BUILD_CONFIG+=" -DUSING_SGDNN_TEST=ON"`不一定有，但是这个子啊默认情况下第一个就有的
		- 然后进入函数，这里会拼接第一个函数`local BUILD_CONFIG="-DNO_TEST=ON -DUSING_MULTI_THREAD_ENGINE=ON -DSOC_MODE=OFF -DUSING_CMODEL=ON -DUSING_BACKEND_API=OFF $1"`
		- 再条件拼接`BUILD_CONFIG+="-DUSING_ONEDNN=OFF "`
		- 再条件拼接`BUILD_CONFIG="$BUILD_CONFIG -DCMAKE_TOOLCHAIN_FILE=${SG1684X_TOP}/toolchain-device.cmake -DSOC_MODE=ON"`，这是交叉编译有值的情况会添加
		- 再新定义一个变量`bin_dir`作为编译成功后的输出路径`local bin_dir=${SG1684X_TOP}/${CMODEL_FW_BINARY_DIR:-build}`
		- 然后将两个变量`BUILD_CONFIG`和`bin_dir`直接传入rebuild_all
		- rebuild_all就是需要参数选项和构建输出路径

