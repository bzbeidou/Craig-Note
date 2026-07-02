```
%input = "top.Input"() {
  shape = [1,3,224,224],
  layout = "NCHW",
  pixel_format = "BGR",
  mean = [104,117,123],
  scale = [0.017,0.017,0.017]
} : () -> tensor<1x3x224x224xf32>

```

```bash
tpuc-opt model.mlir \
  --fuse-preprocess="layout=nhwc, pixel_format=RGB_PLANAR"

```

