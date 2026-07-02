

#### 1、自己写Pass逻辑

这个我们看过很多，一般都是针对ModuleOp或FuncOp的，因为对于模块级以及函数级op都只有一个，不用特别复杂的遍历去获取。

#### 2、使用pattern

这种一般就是针对FuncOp之下的具体的Op。比如add等等

因为对于pattern来说可以直接用apply方法，这个方法能尝试对所有的op应用所写的pattern，省区我们写复杂遍历的过程，很好用

