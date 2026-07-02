### 调用的关系，实现了就是对输入的选项以及输出的路径，实现构建输出的效果

- 实现构建过程

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

# 顺手的事，清空目录
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

