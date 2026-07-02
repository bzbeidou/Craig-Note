```bash
function rebuild_firmware_cmodel()
{
  ## Build firmware_core
  local BUILD_CONFIG="-DNO_TEST=ON -DUSING_MULTI_THREAD_ENGINE=ON -DSOC_MODE=OFF -DUSING_CMODEL=ON -DUSING_BACKEND_API=OFF $1"
  if [ "$DISABLE_ONEDNN" ]; then
    BUILD_CONFIG+="-DUSING_ONEDNN=OFF "
  fi
  if [ ! -z $SOC_CROSS_COMPILE ]; then
    BUILD_CONFIG="$BUILD_CONFIG -DCMAKE_TOOLCHAIN_FILE=${SG1684X_TOP}/toolchain-device.cmake -DSOC_MODE=ON"
  fi

  local bin_dir=${SG1684X_TOP}/${CMODEL_FW_BINARY_DIR:-build} 
  KERNEL_MODULE_PATH=${bin_dir}/firmware_core/libcmodel_firmware.so rebuild_all "${BUILD_CONFIG}" $bin_dir
}
```

### 其中实际上还调用了一个底层

**一般我们就是直接到这里而已**

```cmake
function rebuild_all()
{
  local BUILD_CONFIG=${1:-""}
  local BUILD_FOLDER=${2:-build}

  if [ -d "${BUILD_FOLDER}" ]; then
    rm -r ${BUILD_FOLDER}/*
  fi
  build_all "${BUILD_CONFIG}" "${BUILD_FOLDER}"
}
```

- 回到主目录构建
- `BUILD_CONFIG`
- 输出文件夹就是这里的`BUILD_FOLDER`
- 现其中也是调用的cmake

```cmake
function build_all()
{
  local BUILD_CONFIG=${1:-""}
  local BUILD_FOLDER=${2:-build}

  pushd ${SG1684X_TOP} > /dev/null
  mkdir -p ${BUILD_FOLDER}
  echo "cmake -S . -B ${BUILD_FOLDER} ${BUILD_CONFIG} ${EXTRA_CONFIG}"
  cmake -S . -B ${BUILD_FOLDER} ${BUILD_CONFIG} ${EXTRA_CONFIG} && cmake --build ${BUILD_FOLDER} -- -j$((`nproc`-1));
  ret=$?
  popd > /dev/null
  return $ret
}

```

