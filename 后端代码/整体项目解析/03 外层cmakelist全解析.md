

### 选择添加的子目录

- firmware_core必有
- `backend_api`(USING_BACKEND_API AND(NOT USING_FULLNET))
- USING_CMODEL OR SOC_MODE
- sgdnn_tmp、test/test_layer`文件夹  `USING_SGDNN_TEST AND (CHIP_ARCH STREQUAL "sg2380" )`条件`
- test/test_layerUSING_LAYER_TEST
- USING_PLD_TEST`条件tpuDNN`和`sgdnn_tmp test/test_pld`
-  `tpuDNN、runtime`文件夹 在`USING_TPUDNN AND NOT (CHIP_ARCH STREQUAL "sg2380")`
- `sccl` `USING_TPUDNN`

## git

`include(GitUtils)的内容，加了一个git相关的文件，加载了一些函数功能

- 然后设置一个文件version.c，在build文件夹之下
- 将git版本信息填入这个文件中

```cmake
include(GitUtils)
get_version_from_tag(ver sover REVISION COMMIT_COUNT BRANCH)
set(VERSION_SRC ${CMAKE_CURRENT_BINARY_DIR}/version.c)
# Compile it into binary so we can either:
# nm foo.so | grep tpu1686 to read short commit id from tpu1686_revision_<revision>
# Or compile a simple executable to read tpu1686_revision string if bin is stripped
file(WRITE ${VERSION_SRC}
    "const char *tpu1686_revision =\"${REVISION}\";"
    "const char *tpu1686_revision_${REVISION};")

add_compile_definitions(REVISION="${REVISION}")
add_compile_definitions(COMMIT_COUNT="${COMMIT_COUNT}")
add_compile_definitions(BRANCH="${BRANCH}")
```







