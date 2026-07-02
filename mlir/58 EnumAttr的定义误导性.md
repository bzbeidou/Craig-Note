#### 我们讲了一个EnumAttr的定义，很容易发现他有误导性

```c++
def Module_Chip: I32EnumAttr<"Chip", "module chip",[
      I32EnumAttrCase<"ALL", 0>,
      I32EnumAttrCase<"BM1684",  1000, "bm1684">,
      I32EnumAttrCase<"BM1684X", 1001, "bm1684x">,
      I32EnumAttrCase<"BM1688",  1002, "bm1688">,
      I32EnumAttrCase<"CV186X",  1003, "cv186x">,
      I32EnumAttrCase<"BM1690",  1004, "bm1690">,
      I32EnumAttrCase<"MARS3",  1005, "mars3">,
      I32EnumAttrCase<"SG2380",  1006, "sg2380">,
      I32EnumAttrCase<"SGTPUV8",  1007, "sgtpuv8">,
      I32EnumAttrCase<"CV180x",  2000, "cv180x">,
      I32EnumAttrCase<"CV181x",  2001, "cv181x">,
      I32EnumAttrCase<"CV182x",  2002, "cv182x">,
      I32EnumAttrCase<"CV183x",  2003, "cv183x">
    ]> {
  let genSpecializedAttr = 0;
  let cppNamespace = "::tpu_mlir::module";
}
```
