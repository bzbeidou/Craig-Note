### 使用流程

这些 key 会作为 **Attribute 名** 存放在 `ModuleOp` 上。
 举几个典型的应用场景：

(1) **保存芯片信息**

```c++
module->setAttr(Attr::CHIP, builder.getStringAttr("bm1684x"));
```

在 `.mlir` 文件中会表现为：

```mlir
module attributes { module.chip = "bm1684x" }
```

(2)**保存权重文件路径**

```c++
module->setAttr(Attr::WEIGHT_FILE, builder.getStringAttr("resnet50_weight.npz"));
```

对应 `.mlir`：

```mlir
module attributes { module.weight_file = "resnet50_weight.npz" }
```

(3)保存性能、量化模式、训练还是推理阶段