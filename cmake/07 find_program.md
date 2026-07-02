```
find_program(CCACHE ccache)
```

### find_program(CCACHE ccache)

可执行程序

```
find_program(<变量名> 程序名)
```



```
/usr/bin/ccache
```

变量就是 `CCACHE=CCACHE-NOTFOUND`.

### 

执行：

```
find_program(GIT git)
```

得到：

```
GIT = /usr/bin/git
```

执行：

```
find_program(PY python3)
```

得到：

```
PY = /usr/bin/python3
```

