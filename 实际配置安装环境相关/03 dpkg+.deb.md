```bash
sudo dpkg -i sophon-libsophon_*.deb
```

**这就是一个软件安装的命令**

- 类似于`apt`命令，但是apt就像直接联网应用商店安装，这个dpkg就是使用安装包安装



- **`dpkg`**：Debian 系统的包安装工具
- **`.deb`**：软件包格式，类似于 Windows 的 .msi 或 Red Hat 的 .rpm
- **`-i`**：安装选项
- **`sophon-libsophon_\*.deb`**：Sophon 的运行时库软件包



#### 1、`dpkg` Debian包管理器

**作用**：

- 安装、删除、管理 `.deb` 软件包
- 查询已安装的软件包信息
- 是 `apt` 命令的底层工具



#### 2、**`.deb` - Debian 软件包格式**

这个就是安装包，不同Linux发行版安装阿波的后缀都不一样

| Linux 发行版              | 包格式 | 包管理工具          |
| :------------------------ | :----- | :------------------ |
| **Debian/Ubuntu**         | `.deb` | `dpkg`, `apt`       |
| **Red Hat/CentOS/Fedora** | `.rpm` | `rpm`, `yum`, `dnf` |



### 3、 `dpkg`对比`apt`

```bash
# 直接操作 .deb 文件
sudo dpkg -i package.deb
```

**特点**：

-  不自动解决依赖关系
- 直接安装本地 .deb 文件
- 底层操作

```bash
# 从软件源安装
sudo apt install package-name
```

**特点**：

- 自动解决依赖

	 从网络仓库下载

- 更用户友好



但是apt底层还是`dpkg`，指向apt的时候会经过如下步骤

- 从网上下载对应软件的安装包，就是.deb文件（Debain或Ubuntu的版本）
- 调用`dpkg`来安装这些`.deb`文件