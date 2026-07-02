**关键在于build_backend_lib_cmodel**

```cmake
function build_backend_lib_cmodel()
{
  if [ -z "${BACKEND_BUILD_FOLDER}" ]; then
    BACKEND_BUILD_FOLDER=build # 编译
  fi
  if [ ! -d "$SG1684X_TOP/${BACKEND_BUILD_FOLDER}/" ]; then
    echo "Couldn't find path $SG1684X_TOP/${BACKEND_BUILD_FOLDER}/"
    return -1 #如果目录不存在直接返回报错
  fi

  pushd $SG1684X_TOP/${BACKEND_BUILD_FOLDER}/
  SG1684X_BUILD_CONFIG="-DDEBUG=ON -DUSING_CMODEL=ON -DUSING_BACKEND_API=ON -DUSING_MULTI_THREAD_ENGINE=OFF -DSG_STAS_GEN=ON -DUSING_PLD_TEST=OFF $FAST_GEN_CMD_FLAG $1"
  cores="$(($(nproc)-2))"
  cmake .. ${SG1684X_BUILD_CONFIG} ${EXTRA_CONFIG}; make -j$cores;
  ret=$?
  popd
  if [ $ret -ne 0 ]; then return $ret; fi

  pushd $SG1684X_TOP/$RUNTIME_BUILD_FOLDER/
  SG1684X_BUILD_CONFIG="-DDEBUG=ON -DUSING_CMODEL=ON -DUSING_FULLNET=ON -DUSING_PLD_TEST=OFF $CMODEL_RUNTIME_FLAG $FAST_GEN_CMD_FLAG $1"
  cmake .. ${SG1684X_BUILD_CONFIG} ${EXTRA_CONFIG}; make -j$cores;
  ret=$?
  popd
  if [ $ret -ne 0 ]; then return $ret; fi
}
```

