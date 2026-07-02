#### 1、三种方法

```c++
StringAttr::get(context, name + std::to_string(i))
```

```c++
builder.getStringAttr(name + std::to_string(i))
```

```c++
rewriter.getStringAttr(name + std::to_string(i))
```



#### 2、为什么这里用第三种

专门为写pattern设计的一个类

**至于为什么不用OpBuilder，因为这里是pattern，用这个很统一**