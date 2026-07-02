#### 导出ResNet50 的Onnx模型

如果使用tpu-mlir编译模型，则必须先将Pytorch模型导出为onnx模型。下面以导出1 batch的onnx模型为例进行演示：

```python
import torch
import torch.onnx
from torchvision.models import resnet50

if __name__ == '__main__':
    input = torch.randn(1, 3, 224, 224)          
    model = resnet50()                           # 载入模型框架
    model.load_state_dict(torch.load("xxx.pth"))
    model.eval()                                 # 设置模型为推理模式
    torch.onnx.export(model, input, "xxx.onnx", training=torch.onnx.TrainingMode.TRAINING)  # xxx.onnx表示.onnx文件, 这一步导出为onnx模型, 并不做任何算子融合操作。
```

