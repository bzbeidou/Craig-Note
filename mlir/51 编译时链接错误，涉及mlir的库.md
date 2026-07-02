```c++
add_mlir_library(MyTopPass
  MyTestPass.cpp
  ShapeInfer.cpp
  Init.cpp
  FusePreprocess.cpp

  DEPENDS
  MLIRTopPassIncGen

  LINK_LIBS PUBLIC
  MLIRIR
  MLIRPass
  MLIRShapeInterface
  TPUMLIRSupport
  TPUMLIRBackend
  MLIRQuantDialect
  MLIRQuantUtils
  )
```

