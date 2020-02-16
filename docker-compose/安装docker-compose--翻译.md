[Install Docker Compose 译文](https://docs.docker.com/compose/install/)



### 安装 Docker Compose

###### 你可以在macOS、Windows、64-bit Linux上运行 Compose



### 前提条件

Docker Compose的所有工作都依赖于Docker 引擎，所以你需要确保的安装Compose的位置有安装Docker引擎

- 在Mac和Windows这种桌面系统中，Docker Compose包含在他们的桌面应用中
- 在Linux系统中，首先按照Get Docker Page的描述安装适合你的操作系统版本的Docker，然后回到这里来获得指示信息来安装Docker Compose到你的Linux系统中。
- 如果想以非root用户的方式来使用Compose，请参考[Manage Docker as a non-root user](https://docs.docker.com/install/linux/linux-postinstall/)



### 安装Compose

根据下面的指示信息去安装Compose 到你的Mac、Windows、Windows Server 2016 或 Linux系统中，或者用一些替他可替代的方式来安装Compose，比如使用`pip` 或者安装Compose作为容器。

> 安装不同的版本
>
> 下面的指示信息描述了安装当前的稳定版本（V1.23.4）的Compose。如果想安装不同版本的Compose，请替换你想要的版本的版本号
>
> Compose 的所有版本都可在 [Compose repository release page on GitHub](https://github.com/docker/compose/releases)上获得，如果想安装一个预发布版本的Compose，请参阅 [install pre-release builds](https://docs.docker.com/compose/install/#install-pre-release-builds)章节

---

### Mac 下安装方法

Mac 版的Docker 桌面应用和Docker Toolbox已经包含了Compose，所有Mac用户不必在单独安装Compose了。Docker 关于在Mac下的安装指引如下：

- [Get Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/)
- [Get Docker Toolbox](https://docs.docker.com/toolbox/overview/) (旧系统)



### Linux 下安装方法

在Linux系统中，你可以在[Compose repository release page on GitHub](https://github.com/docker/compose/releases)中下载到Docker Compose的二进制版本。根据链接中的指示信息，你需要在终端中执行`curl`命令来获得二进制文件，下面的手把手的步骤指示也在其中：

> 在`alpine`镜像中，下面列出的依赖包都是需要的: `py-pip`,`python-dev`,`libffi-dev`,`openssl-dev`,`gcc`,`libc-dev`,`make`.

1. 执行下面的命令去下载Docker Compose的最新的稳定版本

   ```shell
   sudo curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```

   > 如果想下载不同的版本，请使用你想要的版本的版本号来替代`1.25.3`

   如果你通过`curl`下载出现了问题，可以看上面的[其他的安装选项](https://docs.docker.com/compose/install/#alternative-install-options)  

2. 提供可执行权限给到二进制文件

     ```shell
sudo chmod +x /usr/local/bin/docker-compose
     ```

> 注意：如果在安装后使用`docker-compose`总是失败，检查一下你的path环境变量，你可以为`/usr/bin`创建一个软连接， 或者添加其他的目录到你的环境变量中。



#### 举例：

```shell
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

1. 可选项，为你的`bash`或`zsh` 安装  [command completion](https://docs.docker.com/compose/completion/)
2. 测试安装

```shell
$ docker-compose --version
docker-compose version 1.25.3, build 1110ad01
```



---

### 安装预发布版本

如果你对尝试预编译版本感兴趣，你可以在 [Compose repository release page on GitHub](https://github.com/docker/compose/releases)下载到它。根据链接中的指引，在终端执行`curl`命令就能下载到其对应的二进制版本

master分支的预发布版本你可以在 https://dl.bintray.com/docker-compose/master/中下载

> 预发布版本允许你体验最新的特性，当可能不那么稳定



### 升级

如果你正想从Compose1.2 或者更早的版本来进行升级，升级后请移除或迁移走你现存的容器。因为在1.3版本后哦，Compose使用Docker labels来追踪容器，所以你需要重建你的容器，并为其添加labels

如果Compose检测到容器容器没有labels，它将拒绝允许，以至于你无法结束它们。如果你希望保留现存的容器（比如：它们拥有你想保存的数据卷），你可以使用Compose1.5版本的如下命令去移植它们：

```shell
docker-compose migrate-to-labels
```

另外，如果你并不关心是否保留这些容器，你可以删除它们，Compose将执行创建新的容器。

```shell
docker container rm -f -v myapp_web_1 myapp_db_1 ...
```



### 卸载

如果你是使用`curl`来安装的Docker Compose，你可以使用如下命令进行卸载

```shell
sudo rm /usr/local/bin/docker-compose
```

如果使用`pip`安装的，使用如下命令卸载：

 ```shell
pip uninstall docker-compose
 ```

> 获得"Permission denied" 错误？
>
> 如果在使用上述方法时收到一个"Permission denied"错误，你可能没有适当的权限去移除`docker-compose`。如果想强制删除，在上述命令前添加 `sudo`, 再执行。

