**实际上这个是在分支里使用，用来表示某种不可能到达的分支，出现了意外情况**

**如果能够正常执行说明没有进入错误分支**

```c++
auto attr = en.value().dyn_cast<IntegerAttr>();
if (attr) {
  data->push_back(attr.getInt());
} else {
  arrayAttr.dump();
  llvm_unreachable("not int64_t type");
}

```
