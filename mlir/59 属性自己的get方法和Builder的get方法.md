- 自己
- Builder

```c++
void setMode(Mode mode_){
  auto s = stringifyMode(mode);
  m->setAttr(Attr::MODE, StringAttr::get(ctx, s));
  
}

void setCoreNum(int64_t core_num){
  m->setAttr(Attr::CORES, Builder(ctx).getI64IntegerAttr(core_num));
}

```

但是我觉得普通属性类提供的get方法也挺好用

在mian函数中我们创建过Builder的对象，但是在这里我们不会复用那个对象，反而是在函数里创建一个Builder的临时对象使用。函数结束后就会被销毁了，并不会造额外的资源损耗，而且逻辑很清晰。

