```bash
function rebuild_test(){
  local TEST_TYPE=${1:-sgdnn} # 如果没传第一个参数
  local TEST_DIR=${2:-build_test} #
  if [ -e "${TEST_DIR}" ]; then 
    rm -r ${TEST_DIR} #就先删除，确保干净构建
  fi

  build_runtime_cmodel || return $?; # 如果失败就返回错误码
  build_test ${TEST_TYPE} ${TEST_DIR} # 调用 build_test 去实际编译测试程序
}




function build_test(){
  local TEST_TYPE=${1:-sgdnn}  # 测试类型
  local TEST_DIR=${2:-build_test}  # 构建目录
  local UPPER_TYPE=`echo $TEST_TYPE | tr 'a-z' 'A-Z'`  # 将测试类型转为大写，例如 sgdnn -> SGDNN
    # 构建 CMake 配置
  local BUILD_CONFIG="-DUSING_CMODEL=ON -DUSING_${UPPER_TYPE}_TEST=ON $FAST_GEN_CMD_FLAG"
  local FIRMWARE_BUILD_CONFIG="-DUSING_${UPPER_TYPE}_TEST=ON"
  if [ -n "$CORE_ID" ]; then
    FIRMWARE_BUILD_CONFIG+=" -DCORE_ID=$CORE_ID"
  fi
  if [ $TEST_TYPE != "cmodel" ]; then
    BUILD_CONFIG+=$CMODEL_RUNTIME_FLAG
  fi

  if [ $TEST_TYPE == "sgdnn" ]; then
    BUILD_CONFIG+=" -DUSING_TPUDNN=ON"
  fi

  if [ $TEST_TYPE == "sgdnn_both_mesg" ]; then
    BUILD_CONFIG+=" -DUSING_SGDNN_TEST=ON"
    FIRMWARE_BUILD_CONFIG+=" -DUSING_SGDNN_TEST=ON"
  fi

  if [[ ($CHIP_ARCH == 'sg2260' || $CHIP_ARCH == 'sg2260e') && $USING_MPI ]]; then
    EXTRA_CONFIG+=" -DUSING_MPI=ON"
    # build_openmpi only download and build openmpi once
    # so it's ok to call build_openmpi multiple times
    build_openmpi
    local PATH=${SG1684X_TOP}/install/bin:$PATH
  fi

  rebuild_firmware_cmodel "${FIRMWARE_BUILD_CONFIG}" || return $?
  ln -sf ${SG1684X_TOP}/build/firmware_core/libcmodel_firmware.so ${SG1684X_TOP}/build/firmware_core/libcmodel.so

  echo "'${BUILD_CONFIG}'"
  build_all "${BUILD_CONFIG}" "${TEST_DIR}" || return $?
  echo "tests is in '${TEST_DIR}'"
}

```

