```c++
op->getAttrOfType<SomeAttrType>("attr_name")
```

tpu-mlir

```c++
auto chip_ = m->getAttrOfType<StringAttr>(Attr::CHIP);
```

返回的指针调用了getAttrOfType这个方法，会去取 `module` 上的属性 `"chip"`

- 如果找到了而且类型匹配，返回 `StringAttr` 对象。
- 如果没找到或者类型不对，返回 `nullptr`。

```
module attributes { chip = "BM1684X" } { ... }
```



复习：在MLIR里的Attribute就似乎类似于键值对字典

有键和值两个内容，**我们这里获取Attribute就是获取对应Attr键的值value**

属性的值可以是以下很多种类型：

- `IntegerAttr`
- `StringAttr`
- `ArrayAttr`
- `BoolAttr`
- `TypeAttr`
- `DenseElementsAtt