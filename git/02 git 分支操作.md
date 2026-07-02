### 1、创建分支

**新创建的分支会保留主分支的内容**

- 只有一个提交（initial commit）
- 所有文件都在这个提交里
- 当前分支：`main`

```
git checkout -b foundation
git checkout -b mlir-study
```

**继承 main 的最新提交**。

可以想象成：

```
main, foundation, mlir-study, backend-study
```

所以**此时所有分支内容完全一样**，foundation, mlir-study, backend-study这几个分支都指向main分支最近的一次提交（也就是在内容上体现为和main分支一样的）。



### 2、创建新分支之后就可以在这些分支新提交内容了，每次提交之后这个分支都会指向一个新的commit指针

举例：

```
# 在 mlir-study 分支修改了 mlir/ 文件
git add mlir/*
git commit -m "mlir-study: add dialect and type notes"

# main 和其他分支仍然保持旧内容
```

所以每个分支的内容会**随着你在该分支上的提交而不同**。



### 3、命令

**创建分支**

```c++
git checkout -b foundation
```

等价于两步，创建并进入分支

```bash
git branch foundation  // 创建分支
git checkout foundation  //进入分支
```





### 4、 切换

**假设现在在foundation分支之下，然后我在工作区修改了一个文件，然后我就不能切换分支了。必须让我add然后commit之后才能切换。**

- 切分支会让工作区的文件内容变成目标分支最新提交的内容
- 如果直接切换，修改的文件就可能被覆盖 → 数据丢失

所以 Git 会报错，阻止切换：

```
error: Your local changes to the following files would be overwritten by checkout:
    somefile.txt
Please commit your changes or stash them before you switch branches.
```

