###[Overview of Docker Compose 译文](https://docs.docker.com/compose/)



##### Docker Compose 是一个用来定义和执行多Docker容器程序的工具，如果使用Compose，你将可以使用一个`YAML`文件来配置你的应用的服务。然后，你可以使用一个单一的命令来读取配置，并创建和启动所有的服务。在学习所有关于Compose的特性之前，你可以看一下 [特性列表](https://docs.docker.com/compose/#features)



##### Compose 可以在所有的环境中运行，例如： 生产，staging，开发，测试 ，就像CI工作流一样。你可以在一些[公共例子](https://docs.docker.com/compose/#common-use-cases)中学习到更多用法。

##### 使用Compose一般都是如下三个步骤：

1. 通过Dockerfile定义你的应用的环境，以至于你可以在任何地方复制它

2. 在`docker-compose.yml`中定义所有组成你的应用的服务，以至于他们可以在一个隔离的环境中一起运行
3. 执行`docker-compose up` 然后compose 会执行并启动你的整个应用



##### 一个`docker-compose.yml`长得像下面这样：

```yaml
version: '3'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

更多的 Compose file 的信息，可以查询 [compose文档](https://docs.docker.com/compose/compose-file/)



Compose 拥有很多管理你的应用的整个生命周期的命令

- 启动、停止 和 重新构建服务
- 观察正在运行的服务的状态
- 流化正在运行的程序输出的日志
- 在服务中执行一个一次性的命令



#### Compose 文档

- [安装 Compose]
- [Compose入门]
- [使用Django入门]



#### 特性

一下这些Compose的特性将非常有用

- 一台宿主机上多环境隔离
- 存储数据卷数据，当一个容器启动时
- 仅当容器发生改变时重新创建
- 在不同的环境中定义变量和组合



#### 一台宿主机上多环境隔离

Compose 使用项目名来隔离不同的环境。你可以在不同的上下文中使用这个项目名称，比如：

- 在一台开发宿主机上，创建多个单独环境的拷贝，比如你想为项目的每个特性分支运行一个稳定的拷贝环境
- 在一台 CI 服务器上，为了防止相互之间的干扰，可以将项目名命为一个唯一的构建数字
- 在一个共享宿主机或开发机上，要去阻止哪些拥有相同的服务名字的项目之间的相互干扰



默认的项目吗时项目的目录的名字，你可以使用`-p`来自定义一个项目名，或者使用`COMPOSE_PROJECT_NAME`来定义一个环境变量



#### 当容器创建时，存储数据卷数据

Compose 存储所有你服务用到的数据卷。当执行`docker-compose up` 时，如果Compose发现之前有可用的容器运行时，会从旧的重启中拷贝起数据卷到新容器中，这个过程时为了确保数据卷中的数据不会丢失。

如果你在windows上使用`docker-compose`，你需要为了你这个特殊的需求而不得不添加一些环境变量，[具体请看这里](https://docs.docker.com/compose/reference/envvars/)



#### 只会在容器发生改变时重建容器

Compose 缓存了用于创建容器的配置。当你重启服务时，如果服务本身没有改变，Compose将复用之间已存在的容器，复用容器意味着你可以快速的改变你的环境



#### 在不同的环境中定义变量和组合

Compose 支持的 `Compose file`中定义变量。你可以用这些变量来为不同的环境或不同的用户做定制。具体请看[变量替换](https://docs.docker.com/compose/reference/envvars/)



你可以通过`extends`字段或者创建多个`Compose file`的方式扩展一个`Compose file`。详情请看[extends](https://docs.docker.com/compose/extends/)



### 通用案例

Compose有很多使用方法。下面概述了一些通用的案例：

#### 开发环境

当你想开发一个软件，让程序运行在一个隔离的环境中的能力和能预期交互的能力至关重要。Compose的命令行工具就拥有上述能力

`Compose file`提供了一种可以归档和配置应用服务的所有依赖（比如： 数据库、队列、缓存、web serivice API等）的方式。使用Compose的命令行工具，你可以静静用一个单独的命令（docker-compose up）就可以为依赖创建并运行一个或多个容器。

这些特性组合在一起就为开发者提供一个非常便利的方式去启动过一个项目。Compose 可以将一个多页的"开发者如门指引"简化为一个机器可读的`Compose file`文件和几个简单的命令。



#### 自动化测试环境

每一一个持续集成或持续部署程序的重要组成部分是自动化测试套件。自动化端到端测试需要一个可以执行这些用例的环境。Compose就提供了一种便利的方式去创建或销毁（为执行你的测试用例的）隔离的测试环境。只要通过Compose file定义好整个环境，就可以可以通过几个简单的命令来创建或销毁这些环境

```shell
docker-compose up -d
./runt_tests
docker-compose down
```



#### 单宿主机部署

Compose 很传统地一直聚焦于开发和测试工作流，但是随着每个版本的发展，我们会提供更多的生产导向的特性。你可以使用Compose来部署到一个远端的Docker引擎。这个Docker引起可能是个由Docker Machine （已淘汰）来控制的单实例，也可能是一整个 Docker Swarm（已淘汰） 集群



#### 发行说明

想查看过去到现在的Docker Compose 的发行版本的更改的详细列表，清查看更改日志



#### 获取帮助

Docker Compose 正在积极发展中。如果你去帮助，比如想做贡献，或者仅仅是想与一些志同道合的人来探讨这个项目，我们有开放一些渠道来用来交流。

- 上报bug或特性请求：使用 issue tracker on Github
- 实时与这个项目的人讨论： 在freenode IRC 上参加docker compose 频道
- 贡献代码或文档的更改：在github上提交一个pull request



更多信息和资源，清浏览 [帮助](https://docs.docker.com/opensource/)

