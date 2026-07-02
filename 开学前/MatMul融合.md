# 一、c++

## 1.1 复习 构造列表

```c++
class Person{
public:
    Person(std::string name, int age) : name(name), age(age){
    }
    
    void speak(){
        std::cout << "hello , I am" << name <<", age" << age << std::endl;
    }
    
    std::string name;
    int age;
}

class Student : public Person{
public:
    Student(std::string name, int age, bool graduated) : Person(name, age), graduated(graduated){};

    bool graduated;
    
}


```



## 1.2 复习 const

```c++
class Foo {
  int x;
public:
  int getX() const {   // 只读函数
    return x;          //  允许读取
  }

  void setX(int v) {   // 非 const 函数
    x = v;             //  可以修改
  }

  void bad() const {
    x = 42;            //  编译错误：const 函数里不能修改成员
  }
};

```



## 1.3 override 

## 1.4 在c++里对指针使用

```
tpu::AddOp a;       // 对象本身
tpu::AddOp* p = &a; // 指针

if (!p) { ... }     // 判断指针是否为空 
if (!a) { ... }     // 对象本身不能直接用 ! 判断

```

#  二 MLIR

## MLIR的类型  

**在mlir里**

 MLIR的类型Type也是用来描述数据的，包括数据形状、存储格式以及元素类型等等，但是在.mlir文件里仅仅是就一个标志

```mli
%0 = "top.Add"(%a, %b) : (tensor<4xf32>, tensor<4xf32>) -> tensor<4xf32>
```

- `(tensor<4xf32>, tensor<4xf32>)`两个输入的类型，`tensor<4xf32>`这个整体就是一个类型，其中分了容器类型、基础类型等

- `-> tensor<4xf32>` → 输出类型
- MLIR 类型可以是：
	- **基础类型**：`i32`, `f32`, `bf16`
	- **容器类型**：`tensor<...>` 表示张量

但是就是字符串表示而已，仅仅是个标志



**到了c++，当mlir文件被读取到c++代码里的时候，**

```mlir
mlir::Type t = op.getOperand(0).getType();
if (auto f32Type = t.dyn_cast<FloatType>()) {//这里强转了基类到子类


}
```

使用 `isa<FloatType>()` 判断类型，`dyn_cast<FloatType>()` 安全转换，`cast<FloatType>()` 强制转换

**对比c++的dynamic_cast工具**

```c++
class Base {
public:
    virtual ~Base() {}
};

class Derived : public Base {
public:
    void foo() { std::cout << "Derived\n"; }
};

Base* b = new Derived();

```

`dynamic_cast` 可以用来 **安全地把 Base转成 Derived**，很简单

```c++
Derived* d = dynamic_cast<Derived*>(b);
if (d) {
    d->foo();  // ✅ 转换成功，可以调用 Derived 方法
} else {
    // 转换失败，d 为 nullptr
}
```



**mlir里的有类似的，之所以没有用c++自带的是因为mlir做了一些优化**

比如这里的isa方法就是判断当前对象的类型的

```c++
Type out_stype = ...;  // mlir::Type 对象
if (out_stype.isa<FloatType>()) {
    // out_stype 真正是 FloatType 类型
}

```

当然mlir也有`dyn_cast<T>`同样的类型转换方法

**为什么 MLIR 需要这种方法？**

- MLIR 的类型系统非常多样：
	- `IntegerType`, `FloatType`, `TensorType`, `MemRefType`，还有用户自定义的 Dialect Type
- 每个 `Type` 对象在内存中都是 **同一个基类 `mlir::Type`**
- 所以运行时要判断：某个 `Type` 对象 **实际是哪种类型**
- 用 C++ 原生 `dynamic_cast` 太慢，MLIR 用 `isa<>`、`dyn_cast<>` 提供高效安全判断和转换。







# 三、TPU_MLIR



##  标准的bias add 

**Bias Add**，就是在矩阵（或张量）运算之后，加上一个 **偏置向量（bias vector）** 的操作。

对于标准的bias add来说，

- bias一定是一维向量
- bias的长度和输入的最后一维长度相当，方便相加

**注意标准的bias一定不是一个常量，而是一个一维向量**

```
Bias: [1, 2, 3]

输出张量（2D）：
[ [a, b, c],
  [d, e, f] ]

广播后的 Bias：
[ [1, 2, 3],
  [1, 2, 3] ]

最终相加：
[ [a+1, b+2, c+3],
  [d+1, e+2, f+3] ]
```







# 四、TPU-MLIR代码解析



## matmul 的 pattern

### 1.1 MatMulWithBias 

**用来综合matmul和bias的**

`MatMulWithBias` 这个优化模式的核心作用就是：

- **识别** MatMul 输出接一个 Add（第二个输入是 1D 向量 bias）的情况；
- **融合** bias 进 MatMul 的第三个输入；
- **删除** 多余的 AddOp。

这样：

1. IR 更简洁（少一个算子）。
2. TPU 硬件映射更高效（MatMul intrinsic 通常自带 bias）。

最终的IR从原本的如下内容

```mlir
%mm = "tpu.MatMul"(%x, %w) : ...
%y  = "tpu.Add"(%mm, %bias) : ...
```

变成

```
%y = "tpu.MatMul"(%x, %w, %bias) : ...
```



### 1.2 MatMulFuseRequant 量化

**原来量化也是Op？**

**当然是Op，任何IR节点（包含算子、量化等等）都是Op**



`MatMulFuseRequant` 的作用如下，可以划分为三个步骤：

**（1）识别模式**

```c++
MatMulOp -> RequantIntOp
```

且 MatMul 输出仅被 Requant 使用。

**（2）融合逻辑**

把 Requant 的 scale/shift/quant_mode/round_mode 属性融合进 MatMul。

生成新的 MatMul, 直接输出量化结果



**举例：**

原始IR

```c++
%mm = "tpu.MatMul"(%x, %w) : (xf32, xf32) -> xi32
%y  = "tpu.RequantInt"(%mm) {multiplier=123, rshift=8} : xi32 -> xi8
```

融合后：

```
%y = "tpu.MatMul"(%x, %w) {multiplier=123, rshift=8, quant_mode=..., round_mode=...} : xi8
```

原本需要先生成 xi32，再 requant，融合后 **MatMul直接输出xi8**。

就像在硬件上用 **MatMul + Requant fusion kernel**。



###  1.3 MatMul2FAttention  针对在attention里的matmul做一定的优化

`MatMul2FAttention` 是一个 pattern，用来识别并把典型的 Transformer Attention 子图（`Q×Kᵀ -> scale -> (add mask) -> softmax -> ×V` 加上各类 `permute/reshape/tile/unsqueeze/cast`）**融合为一个单一的 `tpu::FAttentionOp`（Flash/FAttention）**，前提是满足一系列**数据类型、形状、单用户、硬件与维度限制**。融合能显著减少中间张量、避免重复的转置/reshape/搬运、并调用硬件高效内核。

#### 具体流程解析()的内容



**（1）匹配到的算子链**

从输出端与输入端双向匹配、最后合并为一个 `FAttentionOp`。大体结构：

- **Forward（输出端）**
	 `MatMul2 (Attention-weight × V)` → `Permute(order=[0,2,1,3])` → `Reshape`
	 （这里的 MatMul2 即原来的 `op`）
- **Backward（输入端）**
	 `MatMul0` = `MatMul(Q, K^T)` → `MulConst` (scale = 1/√d) → optional `Add` (bias/mask) → optional `Cast` → `Softmax` → optional `Cast` → MatMul2
- **Queries (Q) 路径**
	 `... -> Permute(q_permute, order=[0,2,1,3]) -> matmul0.left`（必须单用）
- **Keys (K) 路径**（两种常见构造）
	- Case A: `K -> Permute(k_permute) -> (optional Reshape -> Tile/Mul -> Unsqueeze) -> matmul0.right`
	- Case B: `K -> Reshape(k_reshape) -> Tile -> Unsqueeze -> Permute -> matmul0.right`
		 （K 的 permute 顺序根据 `matmul0.getRightTranspose()` 不同有两种接受的顺序）
- **Values (V) 路径**（与 K 对称）
	- `V -> Permute(v_permute) / Reshape(v_reshape)`，如果存在 `Tile/Mul + Unsqueeze`，则**要求 K 也有 tile**（`has_tile` 必须两边一致）

**（2）变量与形状映射**

基于代码里的 `getShape` 与赋值：

- `o_shape = module::getShape(op.getOutput())` —— **最终 Reshape 后的输出形状**
	 代码期望： `o_shape` 是长度 4 的向量：

```
o_shape = [ batch, q_head, mq, d ]
```

即：

- `batch = o_shape[0]`
- `q_head = o_shape[1]`
- `o_shape[2] == mq` （输出的 3rd dim = query 序列长度）
- `d = o_shape[3]` （head_dim）

sf_shape = module::getShape(softmax.getInput())` —— **softmax 的输入 (QK^T 的输出) 的 shape**` 

 期待： `sf_shape = [ batch, q_head, mq, mk ]`


- `mq = sf_shape[2]` （query sequence length）
- `mk = sf_shape[3]` （key sequence length）
- `sf_shape[1] == q_head`（softmax 的第2维为 q_head）

- `kv_shape = module::getShape(k_in)` —— **K (以及 V) 原始输入的 shape**
	 代码使用：
	- `kv_head = kv_shape[2]` （Key/Value 的 head 数）
	- 也要求 `kv_shape == module::getShape(v_in)`（K 与 V 的原始 shape 必须一致）

因此代码做的基本断言：

```c++
o_shape.size() == 4
sf_shape.size() == 4
o_shape[2] == sf_shape[2]      // mq 一致
sf_shape[1] == o_shape[1]      // q_head 一致
module::getShape(k_in) == module::getShape(v_in)
```









#### 搭配代码解析

**1.3.1 确定硬件环境**

只在 **BM1684X / BM1688 / BM1690** 芯片族上启用。

因为 Attention 模式的 GEMM 加速核是硬件特性，不是所有芯片都有。



**1.3.2 检查输出数据类型**

仅支持 **BF16/F16**，不支持 FP32。

这是因为 Transformer 模型推理一般用半精度，TPU 核心也对半精度有特殊优化。



**1.3.3 检查唯一用户约束**

要求 `MatMul` 的输出只有一个下游使用者。

否则，如果融合掉这个 `MatMul`，会影响其它分支的计算路径。



**1.3.4 准备删除的算子列表**

`op_need_del` 记录那些在融合 Attention 模式时可以去掉的辅助算子（比如 reshape、transpose）。

最后 rewrite 阶段会批量 `rewriter.eraseOp`。



**1.3.5 期望的 Attention 模式**

- `MatMul → Permute(0,2,1,3) → Reshape`
- 这是 Transformer 里 **Q×Kᵀ** 或 **Attention 分数矩阵**典型的后处理流程。
- 为什么要这样？
	- Permute 是为了把 `[batch, head, seq, dim]` 转换到 `[batch, seq, head, dim]`。
	- Reshape 通常是为了把多头拼接成一个整体 `[batch, seq, hidden]`。



**1.3.6 严格检查顺序**

- 只接受 `[0, 2, 1, 3]` 的排列，不允许其它 permutation。
- 这是硬件核支持的唯一 Attention 模式。

**1.3.7 唯一使用者约束**

- `Permute` 和 `Reshape` 必须各自只被使用一次，否则优化会破坏其他计算路径。

**1.3.8 记录待删除节点**

- 一旦匹配成功，这些 Permute/Reshape/MatMul 就可以被替换成一个更高效的 **Attention 专用 MatMul op**。

**举例：**

假设有个 Attention 模型 IR 片段：

```mlir
%1 = "tpu.MatMul"(%q, %k) : (tensor<64x128xf16>, tensor<128x64xf16>) -> tensor<64x64xf16>
%2 = "tpu.Softmax"(%1) ...
%3 = "tpu.MatMul"(%2, %v) ...

```

在这里，第一个 MatMul 其实是 **Q×Kᵀ**，符合 Attention 特征。`MatMul2FAttention` 这个 Pattern 就会尝试识别它，把它标记为 "Attention MatMul"，然后在 TPU 上用更高效的 kernel 来跑。



**举例：**

原始 IR

```mlir
%1 = "tpu.MatMul"(%q, %k) : (tensor<1x12x64xf16>, tensor<1x12x64xf16>) -> tensor<1x12x64xf16>
%2 = "tpu.Permute"(%1) {order = [0, 2, 1, 3]} : tensor<1x12x64xf16> -> tensor<1x64x12xf16>
%3 = "tpu.Reshape"(%2) : tensor<1x64x12xf16> -> tensor<1x768xf16>
```

优化后（融合）：

```mlir
%3 = "tpu.MatMul2FAttention"(%q, %k) : (tensor<1x12x64xf16>, tensor<1x12x64xf16>) -> tensor<1x768xf16>

```

