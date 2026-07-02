### 1、先复习一下定义枚举类Attr的方法

- 1 I32EnumAttr 内容是uint_32
- 2 c++枚举类的名字
- 3 简介内容
- 4 I32EnumAttrCase表示
- 5 <"TOP_F32", 0>类的内容

```c++
def Module_State: I32EnumAttr<"State", "module state", [
  I32EnumAttrCase<"TOP_F32", 0>,
  I32EnumAttrCase<"TOP_CALIBRATED", 1>,
  I32EnumAttrCase<"TOP_QUANTIZED", 2>,
  I32EnumAttrCase<"TPU_LOWERED", 3>,
  I32EnumAttrCase<"TPU_REORDERED", 4>,
  I32EnumAttrCase<"TPU_DIVIDED", 5>,
  I32EnumAttrCase<"TPU_ADDRESSED", 6>,
  I32EnumAttrCase<"TOSA_F32", 1000>
]> {
  let cppNamespace = "::tpu_mlir::module";
}

```

（1）枚举

```c++
namespace tpu_mlir {
namespace module {
enum class State : uint32_t {
  TOP_F32 = 0,
  TOP_CALIBRATED = 1,
  TOP_QUANTIZED = 2,
  TPU_LOWERED = 3,
  TPU_REORDERED = 4,
  TPU_DIVIDED = 5,
  TPU_ADDRESSED = 6,
  TOSA_F32 = 1000
};
} // namespace module
} // namespace tpu_mlir
```

（2）辅助

 **工具函数**，方便在 Pass 里用

```c++
llvm::StringRef stringifyEnum(State value);
llvm::Optional<State> symbolizeEnum(llvm::StringRef str);
llvm::ArrayRef<State> getAllEnumValues();
```

### 2、.mlir

```
module attributes { tpu_mlir.module.state = #tpu_mlir.module<State>TOP_F32 }
```

### 3、Attr的写法

**（1）内建类型**

- 例如 `channel_format = "nchw"`

同理整数、浮点数、布尔值

```
%1 = "top.Const"() {value = 42 : i32} : () -> tensor<i32>
```

**（2）自定义**

 TableGen 里定义的 Attribute

- **内建的基础类型（int, float, string, bool） **
- **自定义的复杂 Attr（枚举、struct、array、dict、dialect 特定） **

解析器在看mlir文件的时候就能很快地分清哪些是SSA，哪一部分是函数，哪一部分是Attr，所以mlir的语法设计是和mlir的解析器分不开的