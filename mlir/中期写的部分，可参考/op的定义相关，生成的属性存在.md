**（1）TableGen定义的AddOp**

在TopOps.td种，大致定义如下：

```tablegen
def AddOp : Top_Op<"add", [NoSideEffect]> {
  let arguments = (ins
    AnyTensor:$lhs,
    AnyTensor:$rhs
  );
  let results = (outs
    AnyTensor:$output
  );
}
```

- 二元加法

- lhs和rhs
- 输出 output

**（2）**

生成的.h.inc文件

```c++
class AddOp : public ::mlir::Op<AddOp, ...> {
public:
  using Op::Op;    
  mlir::Value getLhs() { return getOperation()->getOperand(0); }
  mlir::Value getRhs() { return getOperation()->getOperand(1); }
  mlir::Value getOutput() { return getOperation()->getResult(0); }
};
```

- **Wrapper类，**
- 数据都是存储在底层的对象里的
- 拿到底层数据

**（3）Op基类**

```c++
template <typename ConcreteType, typename... Traits>
class Op : public OpState {
public:
    using Op::Op;  
};
```

- `Op<AddOp>`

- 继承自OpState这个类
- 一层Op结构

**（4）OpState**

```c++
class Opstate{
protected:
    Operation *state;
public:
    Opstate(Operation *op) : state(op) {};
    Operation *getOperation() const {return state};
    
};
```

- 内部也就都既有了Operation *state这个指针
- 这个state



**（5）Operation 对象**

```c++
class Operation {
public:
  std::vector<Value> operands; 
  std::vector<Value> results; 
  llvm::StringMap<Attribute> attrs;
 };
```

- Opstate持有的state指针指向它
- 存operands、result、attributes这些



**整体的数据流示意：**

```
AddOp addOpWrapper
+----------------+
| getLhs()       | ----
| getRhs()       |      
| getOutput()    |       
+----------------+        

+-------------------------+
| OpState                  |
| state: Operation* ------>+-
+-------------------------+                       
                                               +----------------+
                                               | Operation      |
                                               | operands[0] -> lhs
                                               | operands[1] -> rhs
                                               | results[0] -> output
                                               | attrs {...}  |
                                               +----------------+
```







