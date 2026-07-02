```c++
  for (auto en : llvm::enumerate(arrayAttr)) {
    auto attr = en.value().dyn_cast<IntegerAttr>();
    if (attr) {
      data->push_back(attr.getInt());
    } else {
      arrayAttr.dump();
      llvm_unreachable("not int64_t type");
    }
  }
```

#### 返回的是什么？

```c++
for (auto en : llvm::enumerate(container)) {
  auto idx = en.index();   // 索引（size_t）
  auto val = en.value();   // 对应的元素（拷贝或引用，取决于 container）
}

```
