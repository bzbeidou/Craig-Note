

### 1、详细推送步骤

（1）**创建为git仓库**

先进入这个文件夹

```bash
cd ~/

git init
```

通过`git init`命令来将此文件夹设置为git仓库

（2）确认当前仓库状态

`git status`

```bash
On branch master (or main)

No commits yet
Untracked files:
  notes/
  README.md
```

（3）加入**暂存区**

**最常用：全部加入`git add .`**



此时再次查看git status`

发现有一些改变需要被提交，也就是刚才add之后把改变后的文件加入了暂存区，但是没有提交到git仓库，下一步需要提交到git仓库

```bash
Changes to be committed:
  new file: README.md
  new file: notes/xxx.md
```

（4）提交commit**

```bash
git commit -m "Inital commit:learning notes"
```

**最重要，经过这一步才将改变的文件提交到git仓库里**

（5）绑定github仓库（只需要第一次提交时绑定）

```bash
git remote add origin git@.。。。
```

**当然这里http地址也可以**

- 验证一下

	```bash
	git remote -v
	```

（7）推送到github

如果是 main 分支

```
git branch -M main
git push -u origin main
```

如果是 master 分支

```
git push -u origin master
```





### 2、典型的git错误（提交冲突）
