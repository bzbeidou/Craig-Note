# 一、Docker

**使用Docker的流程**

```
docker pull = 安装操作系统镜像（只需要一次）
docker run  = 开机进入系统（每次使用时执行）
docker exec = 打开一个终端（像登录）
docker stop = 关机
Docker Desktop = 电源管理系统（必须开着）

```



**什么是镜像，镜像就相当于一个系统安装光盘**

通过pull之后就安装了这个系统，相当于一台新电脑

每次需要进入环境就run，但是run就是创建一个新容器，相当于创建的新用户



可以通过 ps 命令查看当前所有的容器



通过start启动容器等等



## 1. docker容器是一个完整环境

当你 `docker run` 启动一个镜像时，就进入了一个“**完整的 Linux 环境**”。

这个环境里包含：

- Linux 系统（比如 Ubuntu）
- Shell 命令（如 bash、apt、python）
- 你后续安装的所有程序和修改（比如 TPU-MLIR）

相当于全新电脑



## 2. 安装TPU-MLIR

第一次进入容器：

```
docker run -it --name mytpu sophgo/tpu-mlir:latest
```

你现在就在 `mytpu` 这个容器里，里面做的所有操作（clone、编译、安装）都是**保存在这个容器内的**。

二次使用：

不用再 `docker run`，你只要：

```
docker start mytpu
docker exec -it mytpu bash
```

你就回到了**上次关机时的状态**，TPU-MLIR 项目还在！



## 3、涉及命令行的内容

**（1）每次 docker run 就是新建一个容器**

```
docker run -it sophgo/tpu-mlir:latest
```

Docker 会从镜像 `sophgo/tpu-mlir:latest` 新建一个**全新的容器实例**

**（2）关闭**

情况1：你在容器中按下 `Ctrl+C` 或输入 `exit`

- 容器会**自动退出**
- 但状态还保存在 Docker 中（可以看到）

```
docker ps -a   # 能看到 exited 状态的容器
```

情况2：你直接**关闭终端窗口**（不输入 exit）

- 有可能容器还是运行状态（但没终端连接它了）
- 你可以通过 `docker ps` 看到它是否还在运行

情况3：**如果把Docker Desktop关掉**

那么所有容器都会关掉，但是容器内容还存在磁盘上，可以再打开



## 4、windows系统下的docker使用

**（1）命令行提示符和PowerShell**

**PowerShell** 是 Windows 自带的一个更强大的命令行工具，功能比 `cmd` 更丰富，特别适合系统管理和脚本自动化。

**如何打开powershell？**

直接搜索即可

建议使用powershell，只知道他比cmd更强大就行，但是更建议的是使用vscode来进行开发

## 5、docker 常用命令

**（1）查看容器**

```
docker ps
```

输出示例：

```
CONTAINER ID   IMAGE          COMMAND    STATUS        NAMES
a1b2c3d4e5f6   ubuntu:20.04   "bash"     Up 10 mins    brave_wright
```

**查看所有容器**

```
docker ps -a
```

这一步非常重要，因为很多容器运行完会自动退出，你看不到它们，得加 `-a` 才能看到历史记录。

**（2）进入已经创建的容器**

**进入正在运行的容器**

```
docker exec -it <容器ID或者名称> bash
```

示例

```
docker exec -it brave_wright bash
```

这样就会进入容器内部的Linux系统了，像这样：

```
root@a1b2c3d4e5f6:/#
```

**重新进入已经停止的容器**

**先启动容器：**

```
docker start <容器ID或者名称>
```

在进入：

```
docker exec -it <容器ID或者名称> bash
```

**（3）启动/停止/删除 容器**

```
docker start <>
```

```
docker stop <>
```

```
docker rm <>
```



**（4）创建容器**

```
docker run -it --name myubuntu -v ${PWD}:/app ubuntu:20.04 bash
```

| 参数             | 功能                 |
| ---------------- | -------------------- |
| `-it`            | 交互终端             |
| `--name`         | 指定容器名称         |
| `-v ${PWD}:/app` | 挂载当前路径到容器中 |
| `ubuntu:20.04`   | 镜像名称             |
| `bash`           | 启动后的执行命令     |



**（5）实用技巧就是给容器自己定义一个名字，这样方便操作**

**（6）简洁流程**

```
# 1. 创建一个容器并进入交互模式
docker run -it --name myubuntu ubuntu:20.04 bash

# 2. 退出容器（不会停止）
exit

# 3. 再次进入容器
docker exec -it myubuntu bash

# 4. 容器停止了？启动它：
docker start myubuntu

# 5. 查看容器运行状态
docker ps -a

# 6. 删除容器
docker rm myubuntu
```



**（7）pull 下载镜像的**

`docker pull` 命令的作用是从远程的镜像仓库（通常是 Docker Hub）**手动下载镜像**到本地。

```
docker pull <镜像名>:<标签>
```

示例：

```
docker pull ubuntu:20.04
```

意思是从 Docker Hub 拉取名为 `ubuntu`，版本为 `20.04` 的镜像到本地。

**和 `docker run` 的关系**

```
docker run -it ubuntu:20.04
```

这条命令**会自动执行以下流程**：

1. 检查你本地有没有 `ubuntu:20.04` 镜像；
2. 如果没有，就自动执行 `docker pull ubuntu:20.04`；
3. 有镜像后，再执行 `docker run`。

所以你**可以不写 `docker pull`，直接 `docker run`，Docker 会自动帮你拉镜像**。

**想查看本地已经拉取下来的镜像**

```
docker images
```

想清理不需要的镜像

```
docker rmi <镜像ID或名>
```

**（8）补充：命令中的-it bash**

```
docker exec -it <容器ID或者名称> bash
```

`-it` 是什么意思？

`-it` 实际上是两个参数的组合：

| 参数 | 全称            | 作用                                                      |
| ---- | --------------- | --------------------------------------------------------- |
| `-i` | `--interactive` | 保持容器的标准输入（stdin）处于打开状态，允许你输入命令   |
| `-t` | `--tty`         | 为容器分配一个伪终端（TTY），让你获得一个“命令行界面”体验 |

 换句话说，`-it` 就是为了让你**能够“进入”容器，像在 Linux 系统里一样操作命令**。

```
docker exec -it myubuntu bash
```

如果没有 `-it`，你就只能看到一闪而过的输出，无法交互。

而加了 `-it`，你就能进入这个容器，看到：

```
root@a1b2c3d4:/#
```

这时你就可以像用 Linux 那样输入命令

# 二、 如何访问到容器环境

```
$ docker run --privileged --name myname -v $PWD:/workspace -it sophgo/tpuc_dev:v3.4
```

在run时，相当于安装这个系统，但是我们要怎么用这个系统呢，这里就用到了-v这个方法

$PWD是指的当前我们打开powershell的文件夹

/workspace则是容器内的一个文件夹，这样就相当于把两个文件夹联通了，写在本的目录下的文件同时会出现在container下的/workspace文件夹之下

**流程：**

1、在电脑上创建一个项目目录：

```
C:\Users\Tomking\my-tpu-project
```

2、用 PowerShell 进入这个目录：

```
cd C:\Users\Tomking\my-tpu-project
```

3、启动容器并挂载当前目录进去（这一步是必须的，这样容器就绑定了当前这个文件夹）：

```
docker run --privileged --name tpu-container -v ${PWD}:/workspace -it sophgo/tpuc_dev:v3.4
```

4、你进入容器后，默认在 `/workspace` 目录下，它等价于你电脑的 `my-tpu-project` 目录；

5、所以你只要把模型、代码、测试数据放在你本地的那个目录，容器中就能用了。



**推荐在run时挂载，后面也可以用其他方法，但是麻烦**
