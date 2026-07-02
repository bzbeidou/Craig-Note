#### 1、python类的继承

**python**

```py
# 定义地RMSNorm类继承了torch.nn.Moudule类
# 而且在构造函数里必须显式调用super().__init__()
# 因为super()是一个函数，返回父类对象，然后链式调用父类__init__()方法
class RMSNorm(torch.nn.Module):
    def __init__(self, dim: int, eps: float = 1e-6):
        super().__init__()
        self.eps = eps
        self.weight = nn.Parameter(torch.ones(dim))
```



#### 2、`__init__` ，`__call__`这种方法是什么

这是叫做魔法方法，必须由__包裹，**语法糖配合使用**

**也就是语法糖背后所调用的方法都是魔法方法**

```python
class Greeter:
    def __init__(self, name):
    	self.name = name
    def __call__(self, name):
        print(f"Hello, {name}!")

g = Greeter()  # 这里用语法糖自动调用了__init__方法
g("Tom")  # 用语法糖自动调用 __call__方法

```

还有好多Magic Method，举例**

| 魔法方法              | 自动触发时机            | 等价的“糖”写法    | 解释                   |
| --------------------- | ----------------------- | ----------------- | ---------------------- |
| `__init__(self, ...)` | 创建对象时自动调用      | `obj = MyClass()` | 构造方法，初始化属性   |
| `__call__(self, x)`   | 对象像函数一样被调用时  | `obj(x)`          | 让对象“像函数一样”使用 |
| `__str__(self)`       | `print(obj)` 时自动调用 | -                 | 决定打印出来的字符串   |



#### 3、RMSNorm的具体实现

模拟一下一个2*4的张量

```
x = torch.tensor([[1.0, 2.0, 3.0, 4.0],
                  [2.0, 4.0, 6.0, 8.0]])

```

**如何经过RMSNorm？**

```python
norm = RMSNorm(dim=4, eps=1e-6) # 创建对象
output = norm(x) #调用__call__()方法，Module里有一个__call__()方法内调用了forward()方法



```

```py
self.weight = nn.Parameter(torch.ones(dim))
```

**详细讲解进入forward方法之后**

```py
#对第一行[1,2,3,4]
#1 平方
[1^2, 2^2, 3^2, 4^2] = [1, 4, 9, 16]

#2 平均
mean = (1 + 4 + 9 + 16) / 4 = 7.5


scale = 1 / sqrt(7.5 + 1e-6) ≈ 0.3651

[1, 2, 3, 4] * 0.3651 ≈ [0.3651, 0.7302, 1.0953, 1.4604]

最终输出： [0.3651, 0.7302, 1.0953, 1.4604]

对第二行 [2, 4, 6, 8]
1、平方
[4, 16, 36, 64]
2、均值
(4 + 16 + 36 + 64) / 4 = 30.0
3、scale
1 / sqrt(30.0 + 1e-6) ≈ 0.1826
4、输入 * scale
[2, 4, 6, 8] * 0.1826 ≈ [0.3651, 0.7302, 1.0953, 1.4604]

```

当两个形状（shape）不同的 Tensor 做运算时，PyTorch 会自动把其中 **形状较小的 Tensor** 复制扩展（不是内存上真的复制，而是逻辑上扩展），使它们形状一致

```
1.从后往前对齐两个张量的 shape；
2.如果对应维度(3种情况)：
相等，可以广播

或其中一个是 1，可以广播
```

```py
#例1
A = torch.rand(3, 4)
B = torch.rand(1, 4)
C = A + B  # 自动把B扩展成 (3, 4),这里就是语法糖，实际上调用了一定方法脱产B

# 先看后面维度是4，继续往前发现维度不一样，但是有一个是1，对1的维度进行拓展到3，完全一致
```

```py
# 例2
A.shape = (2, 3, 4)
B.shape = (1, 4)

从后往前看：
4 对 4 ✅
3 对 1 ✅ → 广播
2 对 空位 ✅ → 视为1 → 广播

结果 shape = (2, 3, 4)

```

```py
A.shape = (3, 4)
B.shape = (2, 4)

→ 第一个维度 3 和 2 无法广播 ❌

```



```python
A * B  # shape 必须一样，或能广播成一样

例：
A = torch.tensor([1, 2, 3])
B = torch.tensor([10, 20, 30])
C = A * B  # 结果: tensor([10, 40, 90])

```

```py
A = torch.tensor([[1, 2], [3, 4]])  # shape: (2, 2)
B = torch.tensor([[5, 6], [7, 8]])  # shape: (2, 2)
C = torch.matmul(A, B)             # 或 A @ B

# 结果:
# [[1*5 + 2*7, 1*6 + 2*8],
#  [3*5 + 4*7, 3*6 + 4*8]]
# => [[19, 22], [43, 50]]

```



#### 位置编码 ----解决自注意力机制没有顺序的问题

**关于位置编码直接看知乎文章[Transformer学习笔记一：Positional Encoding（位置编码） - 知乎](https://zhuanlan.zhihu.com/p/454482273)**  ，讲的很清楚，**所看到的PEt实际上都是一个向量**

**直接自己画个图，有多少维度就有多少个正弦波，然后每个正弦波的频率就由维度的位置来确定，那么token的位置k就可以看作是看作是变量，画出来就是竖着的dim个正弦波**

**比如输入是(batch, seq_len, dim)是（1，3，10），也就是3句话，每句话10个单词，那么可以画出10条不同频率的正弦波**

至于公式里为什么要选择10000，是因为需要让频率尽可能的小，这样可以保证每个位置向量都不相同

这点文章里也有讲

引入cos则是为了使其每个位置向量之间能够线性变换

传统位置编码就是Transformer架构论文中提出的

```py
# 简单代码示例
def positional_encoding(seq_len, dim):
    pe = torch.zeros(seq_len, dim)
    position = torch.arange(0, seq_len).unsqueeze(1)
    div_term = torch.exp(torch.arange(0, dim, 2) * (-math.log(10000.0) / dim))
    pe[:, 0::2] = torch.sin(position * div_term)
    pe[:, 1::2] = torch.cos(position * div_term)
    return pe
```

```
解析
torch.zeros(seq_len, dim)创建一个为0的二维向量
torch.arange(0, seq_len)创建一个一维向量[0,1,2...,seq_len - 1]
.unsqueeze(1)这个就是拓展维度用的，看下面详细解析
```

**unsqueeze(0)和unsqueeze(1)  用于拓展维度，torch的方法**

```py
#就是从一维向两维拓展,本来是仅仅是一维的一个数组，用了unsqueeze之后才变成二维

x = torch.arange(5)
print(x)
# tensor([0, 1, 2, 3, 4])
# shape: (5,) → 1维张量（向量）

x.unsqueeze(0)
# shape: (1, 5) → 变成 2D，1 行 5 列 [[0,1,2,3,4]]
x.unsqueeze(1)
# shape: (5, 1) → 变成 2D，5 行 1 列
[
    [0],
    [1],
    [2],
    [3],
    [4]
]
```



```py
div_term = torch.exp(torch.arange(0, dim, 2) * (-math.log(10000.0) / dim))
# 这里就是在构造频率系数，通过torch.arange(0, dim, 2)引入dim//2个系数，对应公式的频率系数部分
# 最终得到的是一个一维数组，一维向量
```

```py
# 直接用
position * div_term

# 广播机制派上用场了
# 一个是(seq-1, 1),一个是(dim//2)
# 变为(seq-1, dim//2)，然后变为(seq-1,dim//2)，这样就能对应相乘了

#如下
position:     [[0],       →  [[0, 0, 0],
               [1],            [1, 1, 1],
               [2]]            [2, 2, 2]]

div_term:     [0.1, 0.2, 0.3]  →  [0.1, 0.2, 0.3]
                          ⬇️广播
乘法结果：
[[0*0.1, 0*0.2, 0*0.3],      → [[0.0, 0.0, 0.0],
 [1*0.1, 1*0.2, 1*0.3],         [0.1, 0.2, 0.3],
 [2*0.1, 2*0.2, 2*0.3]]         [0.2, 0.4, 0.6]]

```

```py
pe[:, 0::2] = torch.sin(position * div_term)
pe[:, 1::2] = torch.cos(position * div_term)
return pe;
```



```py
# 例
pe = torch.tensor([
    [10, 11, 12, 13, 14, 15],
    [20, 21, 22, 23, 24, 25],
    [30, 31, 32, 33, 34, 35]
])
pe[:, 0::2]
得到：
tensor([
    [10, 12, 14],
    [20, 22, 24],
    [30, 32, 34]
])
```

```py
a[start : stop : step]
# start: 起始索引
# stop: 结束索引（不含）
# step: 步长
```



#### llama2中的旋转位置编码  

**要求：dim维度是偶数，因为RoPE就是把token的最后一维dim按两个一组分成复数向量，一个实数、一个虚数**

```
原始向量: [x₀, x₁, x₂, x₃, ..., x_{d-2}, x_{d-1}]
⟶ 分组:    [(x₀,x₁), (x₂,x₃), ..., (x_{d-2}, x_{d-1})]
```



**看下面RoPE的核心代码**

```py
def precompute_freqs_cis(dim: int, end: int, theta: float = 10000.0):
    freqs = 1.0 / (theta ** (torch.arange(0, dim, 2).float() / dim))  # dim/2 频率值
    t = torch.arange(end)  # 位置0到end-1
    freqs = torch.outer(t, freqs)  # shape: (end, dim//2)
    freqs_cis = torch.polar(torch.ones_like(freqs), freqs)  # 复数形式 e^{iθ}
    return freqs_cis
```

**举例说明使用方式（dim = 4， end = 3）**

**第一步：准备频率**

```python
dim = 4;
theta = 10000
torch.arange(0, 4, 2) = [0, 2]
freqs = 1 / (10000 ** ([0, 2] / 4)) = [1, 1 / 100]
# 为什么这里得到的频率是一个一维向量，因为我们dim是4，两两一组分成2组，那么每一组要有不同的频率
# 类似于传统位置编码中，dim中的每个i都对应着一个频率，那么这里每一组也要有不同的频率

```

**第二步：生成位置**

```py
t = [0, 1, 2]
```

**第三步：outer生成角度矩阵**

```
freqs_outer = outer(t, freqs) = 
[[0 * 1,   0 * 0.01],       → [0.00, 0.00]
 [1 * 1,   1 * 0.01],       → [1.00, 0.01]
 [2 * 1,   2 * 0.01]]       → [2.00, 0.02]

```

**第四步：得到复数矩阵freqs_cis**

```python
torch.polar(1, angle) = cos(angle) + sin(angle) * i
→ freqs_cis 是 shape (3, 2) 的复数矩阵，对应每个位置的旋转角度
```

**第五步：将原始张量转换成复数，然后与上面的复数矩阵相乘**





*结合实际Q、K应用解释*

假设你的 query 向量是 `x=[x1, x2, x3, x4]`，我们将它两两一组视为复数：(x1 + j*x2), (x3 + j*x4)，然后将它**乘上一个单位复数 e^(jθ)**，这就是一个旋转操作。

```
(x1 + jx2) * e^(jθ) = (x1 cosθ - x2 sinθ) + j(x1 sinθ + x2 cosθ)
```

我们给每一维设置不同的旋转角度（随位置变化），这样模型在 self-attention 中就能“感受到”位置的变化。

**继续举例上面dim = 4，end = 3的情况**

那么：

- `freqs = torch.arange(0,4,2) / 4 = [0, 0.5]` → `[1.0, 1/√10000]`
- `t = [0,1,2]`

然后得到：

```
freqs_cis = [
    [e^(j*0),        e^(j*0)],        # 第0个位置
    [e^(j*θ1),       e^(j*θ2)],       # 第1个位置
    [e^(j*2θ1),      e^(j*2θ2)],      # 第2个位置
]

```

每个位置有一组旋转角度，最终你会把 Query 向量（Q）变成复数乘上对应的旋转：

```
xq_ = torch.view_as_complex(xq.reshape(...))
xq_out = torch.view_as_real(xq_ * freqs_cis).flatten(...)

```



#### llama2中的attention_mask

用来屏蔽无用的token，防止看到未来token

在 ONNX 图中，LLaMA2 使用的是 **causal mask**（因果掩码），它会在计算 Attention 分数（Q·Kᵀ）之后加上一个大负数（通常是 -1e9）来屏蔽未来的位置。

**在onnx格式图中可以很明显在Q·Kᵀ之后加了一个数值，这个数就是在attention_mask输出的**

```
Attention中的计算流程（）
Q = Linear(x)
K = Linear(x)
V = Linear(x)

scores = (Q @ K^T) / sqrt(d)
scores = scores + attention_mask   ← 在这一步加 mask
probs = Softmax(scores)
output = probs @ V

```

