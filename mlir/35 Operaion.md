

```c++
    auto returnOp = dyn_cast<ReturnOp>(entryBlock.back()).getOperation();
    for (uint32_t i = 0; i < returnOp->getNumOperands(); ++i) {
      returnTypes.push_back(returnOp->getOperand(i).getType());
    }
```

**重点：有一个Operation的指针作为成员变量存在每个Op里**

- 所以`getOperation`是拿出当前Op底层存储的成员变量指针
- `getNumOperands()、getOperand(i)`而不是对应Op类的方法，这也很明显，所以方法在这个类里很明显



- **先判断当前的算子是不是**

- **是的话就获取到它底层**

- **调用这些方法**

- **获取到相关数据**