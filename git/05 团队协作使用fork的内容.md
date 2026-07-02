### 1、 fork是什么

团队协作的经典形式

- 已有一个项目，但是我没有这个项目的修改权
- 我可以fork这个项目（在我的仓库创建了同样的项目）
- 再通过提交pr请求到原仓库
- 原仓库管理员审核无误后通过，合并

**这是fork**

### 2、分支问题

**用fork的方式很正确**

- 但是在自己的仓库里也要创建分支

**整体流程：**

- 在自己fork的仓库上创建新分支（就是新功能），一般叫做feature-rsqrt这种
- 然后在这个feature分支上把这个功能写完，push到自己的仓库上
- 然后提交pr申请
- 原仓库管理员通过之后会合并到主分支main
- 这时候我们再pull原仓库到我们自己本地的main分支
- 此时feature-rsqrt已经可以删除了，因为其内容已经merge到main了
- 然后再创建新分支开发新功能，这样循环往复



### 3、设置upstream

**问题是我们要pull的话需要拉取原仓库的内容，因为是管理员审核合并后的内容**

**所以最好设置一个upstream来代替原仓库地址，这样方便，不用每次输入原仓库地址**

```bash
git remote add upstream 原仓库地址
```

```
git pull upstream main
```



**pull到本地之后再push自的github仓库即可，然后可以删掉分支**

- 注意除了从原仓库pull之后需要push到自己github main分支，其他情况push到github的时候都是push到其他功能分支





