```bash
bin/my-opt --fuse-preprocess --customization_format="RGB_PLANAR" ../yolov5s.mlir -o ../yolov5s_preprocess.mlir

```

```bash
bin/my-opt --fuse-preprocess="--customization_format=RGB_PLANAR" ../yolov5s.mlir -o ../yolov5s_preprocess.mlir

```



#### 1、用Pipeline Pass

```bash
bin/my-opt --pass-pipeline="builtin.module(fuse-preprocess{customization_format=RGB_PLANAR})" ../yolov5s.mlir -o ../yolov5s_preprocess.mlir

```

 - `builtin.module(...)`：指定 pipeline 的入口是 ModuleOp。
  - `fuse-preprocess{customization_format=RGB_PLANAR}`：你的 Pass 及选项。
  - `../yolov5s.mlir`：输入文件。
  - `-o ../yolov5s_preprocess.mlir`：输出文件。

#### 2 具体

```bash
bin/my-opt --fuse-preprocess ../yolov5s.mlir -o ../yolov5s_preprocess.mlir
```

#### 3 如何使用Pipeline

```c++
bin/my-opt --pass-pipeline="builtin.module(fuse-preprocess)" ../yolov5s.mlir -o ../yolov5s_preprocess.mlir
```

**（2）Pipeline的选项**

```bash
bin/my-opt --pass-pipeline="builtin.module(fuse-preprocess{customization_format=RGB_PLANAR mode=INT8 align=true})" ../yolov5s.mlir -o ../yolov5s_preprocess.mlir
```

**（3）Pipeline里运行多个Pass**

```c++
bin/my-opt --pass-pipeline="builtin.module(fuse-preprocess{customization_format=RGB_PLANAR},another-pass)" ../yolov5s.mlir -o ../yolov5s_preprocess.mlir
```
