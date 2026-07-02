### cmake版本警告问题  了解就行

cmake一直会更新，因此其中的一些命令可能会有小改动，所以会出现版本的警告问题，可以限制某个版本为老版本`OLD`,如果设置新版本则是`NEW`

```cmake
if (POLICY CMP0116)
  cmake_policy(SET CMP0116 OLD)
endif()

if (POLICY CMP0148)
  cmake_policy(SET CMP0148 OLD)
endif()

```



