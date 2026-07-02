类比NVIDIA的开发方式，对比tpu-mlir和libsophon这两种配合的关系。

## NVIDIA 生态 vs Sophon 生态对比

下面是一些详细对比，可以看到这种**主机驱动+容器化工具链的模式**已经成为AI加速卡的标准架构

- **NVIDIA**: Driver + CUDA Docker images
- **Intel**: GPU Driver + OpenVINO Docker
- **AMD**: ROCm Driver + PyTorch Docker
- **Sophon**: libsophon + tpu-mlir Docker



### 1、驱动层

```bash
# 安装在主机上
sudo apt install nvidia-driver-550
nvidia-smi  
```

```bash
# 安装在主机上,这部分是提前下载好的内容，而不是通过apt工具来下载的
sudo dpkg -i sophon-driver_*.deb
bm-smi    
```



### 2、运行时库对比

```bash
# CUDA Toolkit (包含运行时库)
sudo apt install cuda-runtime-12-2
```

```bash
# libsophon 运行时,这里也是下好的，这个和上面那个deb文件都是属于libsophon的内容
sudo dpkg -i sophon-libsophon_*.deb
```



### 3、开发工具链对比

```bash
# 在 Docker 中使用 CUDA 开发环境
docker run --gpus all -it nvidia/cuda:12.2.0-devel-ubuntu20.04
# 在容器内使用 nvcc 编译 CUDA 代码
nvcc -o program program.cu
```

```bash
# 在 Docker 中使用 tpu-mlir 环境
docker run --device /dev/bmdev-ctl -it sophgo/tpu-mlir:latest
# 在容器内使用 tpu-mlir 编译模型
tpu-mlir --model your_model.onnx --output your_model.bmodel
```

