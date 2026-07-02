#### 1、镜像管理

`docker images`  列出本地镜像



#### 2、容器管理

`docker run nginx`  运行容器

`docker ps`  查看所用容器



`docker start container_name` 启动容器

`docker stop container_name` 停止容器



`docker exec -it container_name /bin/bash` 进入容器

| 进入方式          | 退出命令             | 容器状态     |
| :---------------- | :------------------- | :----------- |
| `docker exec -it` | `exit` 或 `Ctrl + D` | **继续运行** |

使用 `exec` 进入容器后的退出方式

安全退出（推荐）

```bash
# 退出并保持容器运行
exit
# 或者
Ctrl + D
```

