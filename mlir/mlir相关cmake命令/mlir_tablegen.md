### 使用方法

```
mlir_tablegen <input.td> -gen-<backend> -o <output-file>
```

其中：

- `<input.td>` ：TableGen 文件（定义 Dialect、Op、Attr、Type 等）
- `<backend>` ：指定生成的内容
- `<output-file>` ：输出的 .h 或 .cpp

| 选项                 | 作用                          | 示例输出文件        |
| -------------------- | ----------------------------- | ------------------- |
| `-gen-op-decls`      | 生成 Op 的 **类声明**（.h）   | `TopOps.h`          |
| `-gen-op-defs`       | 生成 Op 的 **类实现**（.cpp） | `TopOps.cpp`        |
| `-gen-dialect-decls` | 生成 Dialect **声明**（.h）   | `TopDialect.h`      |
| `-gen-dialect-defs`  | 生成 Dialect **实现**（.cpp） | `TopDialect.cpp`    |
| `-gen-attrdef-decls` | 生成 Attribute **声明**       | `TopAttributes.h`   |
| `-gen-attrdef-defs`  | 生成 Attribute **实现**       | `TopAttributes.cpp` |
| `-gen-typedef-decls` | 生成 Type **声明**            | `TopTypes.h`        |
| `-gen-typedef-defs`  | 生成 Type **实现**            | `TopTypes.cpp`      |
| `-gen-pass-decls`    | 生成 Pass **声明**            | `Passes.h.inc`      |
| `-gen-pass-defs`     | 生成 Pass **注册/实现**       | `Passes.cpp.inc`    |

CMake 宏来调用，比如：

```cmake
set(LLVM_TARGET_DEFINITIONS TopOps.td)
mlir_tablegen(TopOps.h.inc -gen-op-decls)
mlir_tablegen(TopOps.cpp.inc -gen-op-defs)
add_public_tablegen_target(TopOpsIncGen)
```

`LLVM_TARGET_DEFINITIONS`：指定 `.td` 文件

`mlir_tablegen`：调用工具生成 `.inc` 文件

`add_public_tablegen_target`：生成一个 target（比如 `TopOpsIncGen`

实际上，在这句代码里`mlir_tablegen(TopOps.h.inc -gen-op-decls)`就已经调用mlir_tablegen工具生成了.inc文件内容，已经可以用来，为什么后面又而外添加了一句呢

这个过程注册为一个target，名为TopOpsIncGen

确保了.inc文件已经生成

```cmake
add_mlir_dialect_library(MyDialect
  TopDialect.cpp
  TopOps.cpp
  DEPENDS
    TopOpsIncGen  
)
```
