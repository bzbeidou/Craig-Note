`docker ps -a` ps 就是processing status， 会呈现当前所有的容器状态

`docker images`  查看docker内装的全部镜像信息

`docker system df`   system代表Docker系统管理相关的命令，df就是disk free的简写，来自linux传统，用来查看占了多少磁盘空间 查看当前docker的镜像和容器占用空间的信息







#### 为什么这里的命令中有的没写-或--，反而有的写了呢？

因为 **命令不要写，但是命令的选项需要有-或者--**

而且docker是使用的子命令的模式，基本上很多额外下载的内容都是使用这个模式的



#### 1、传统选项模式  就是Unix的传统

基本结构：`命令 【选项】 【参数】`

**比如：**

下面这些命令模式都很统一，

- 一个命令`ls`
- 一个或多个选项`-a`
-  然后可以有参数，比如/home

```bash
ls -a /home
tar -xzvf archive.tar.gz
```



**这种当然很清晰，但是如果说一个命令里的功能增多时，选项就变得更多，而且选项常用缩写，所以难以记忆，因此有了下面的子命令模式**





#### 2、子命令模式(Subcommand Style)

基本结构：`命令 子命令 【选项】 【参数】`

比如：

以`docker`为例，其下有很多子命令，比如`container`和`system`，然后后面才是选项，这样相关功能分组在一起。这里ls尤其是子命令，不要搞错了，其实就是相当于划分文件夹

```bash
docker container ls -a
docker system df
git commit -m "message"
```



