## 完整命令

```
git remote add origin git@github.com:yourname/learning-notes.git
```

- **git** → 调用 Git 工具
- **remote** → 管理远程仓库的命令
- **add** → 添加一个新的远程仓库
- **origin** → 给这个远程仓库取的名字（可以随便起，但习惯叫 origin）
- **git@github.com:yourname/learning-notes.git** → 远程仓库的地址（SSH 形式）

------

## 解释

| 部分                    | 作用     |
| ----------------------- | -------- |
| `git remote`            | 查看     |
| `add`                   | 添加     |
| `origin                 | 起个名字 |
| `git@github.com:`。。。 | 远程仓库 |



## 举例

添加远程仓库：

```
git remote add origin git@
```

查看远程仓库：

```
git remote -v
```

输出：

```
origin  git@github.com: (fetch)
origin  git@github.com: (push)
```



## 注意

- 如果你之前已经添加过 origin，再次执行 `git remote add origin ...` 会报错
- 修改远程地址可以用：

```
git remote set-url origin git@github
```

