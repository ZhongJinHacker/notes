在这一页，你将学习到如何构建一个简单的python的web应用，并通过Docker compose来运行。这个应用程序使用的是`Flask`框架，并维护着一个存储在reids里的点击计数器。由于这个案例使用的是python，所以你其中的一些概念你必须了解，即使你对它不是很熟悉。



### 前提条件

确定你已经安装了 Docker 引擎和Docker Compose。你不必安装python和Redis，它们将有Docker引擎提供。



### 第一步：设置

定义应用的依赖。

1. 为项目创建一个目录

   ```shell
   $ mkdir composetest
   $ cd composetest
   ```

2. 在项目目录下创建`app.py`的文件，并粘贴下面的内容：

   ```shell
   import time
   
   import redis
   from flask import Flask
   
   app = Flask(__name__)
   cache = redis.Redis(host='redis', port=6379)
   
   
   def get_hit_count():
       retries = 5
       while True:
           try:
               return cache.incr('hits')
           except redis.exceptions.ConnectionError as exc:
               if retries == 0:
                   raise exc
               retries -= 1
               time.sleep(0.5)
   
   
   @app.route('/')
   def hello():
       count = get_hit_count()
       return 'Hello World! I have been seen {} times.\n'.format(count)
   ```

   在这个案例中，`redis`是应用网络中的redis容器的主机名。我们使用的是redis的默认端口号`6379`。

   > 处理临时错误
   >
   > 注意`get_hit_count`函数的实现。这个基本的重试循环让我们能在redis服务不可用时多次重试我们的请求。当我们的应用上线时，这个操作十分有用。而且会使我们的应用更有弹性，当redis服务在应用生命周期内需要频繁重启时。在一个集群中，这也帮助我们处理节点间链接瞬时断开的场景。

3. 在项目目录中创建 `requirements.txt`文件，并粘贴一下内容：

   ```shell
   flask
   redis
   ```

### 第二步：创建Dockerfile文件

在这一步，你将编写一个用于构建Docker镜像的Dockerfile文件。这个镜像包含这个Python应用说需要的所以依赖，并包含python本身。

在你的项目目录下，创建一个名为`Dockerfile`的文件，然后粘贴以下内容：

```shell
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run"]
```

这相当于告诉Docker要做以下事情：

- 根据Python3.7镜像来构建这个镜像
- 设置工作目录为`/code`
- 设置环境变量给`flask`命令使用
- 安装gcc的so文件、python 包（比如：MarkupSafe和 SQLAIchemy）用于编译加速
- 复制`requirements.txt`然后安装python依赖
- 复制宿主机当前目录`.`到镜像的工作目录`.` 下
- 为容器设置默认命令`flask run`

想获得更多关于如何编写Dockerfiles的信息，请参阅 [Docker user guide](https://docs.docker.com/engine/tutorials/dockerimages/#building-an-image-from-a-dockerfile) 和  [Dockerfile reference](https://docs.docker.com/engine/reference/builder/).



### 第三步： 在Compose file 中定义服务

在项目中创建一个叫`docker-compose.yml`的文件，然后粘贴以下内容：

```shell
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

这个Compose 文件定义了两个服务：`web`和`redis`。

#### Web 服务

这个`web`服务使用之前在当前目录的Dockerfile构建出来的镜像。它将绑定及暴露容器和宿主机的`5000`端口。这个案例服务使用的是Flask web服务的默认端口`5000`。

#### Redis 服务

这个`redis`服务使用的是Docker Hub 库中的公开的 Redis镜像。



### 第四步：通过Compose构建并运行你的应用

1. 在你的工程目录下，使用`docker-compose up`来启动你的应用

   ```shell
   $ docker-compose up
   Creating network "composetest_default" with the default driver
   Creating composetest_web_1 ...
   Creating composetest_redis_1 ...
   Creating composetest_web_1
   Creating composetest_redis_1 ... done
   Attaching to composetest_web_1, composetest_redis_1
   web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
   redis_1  | 1:C 17 Aug 22:11:10.480 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
   redis_1  | 1:C 17 Aug 22:11:10.480 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=1, just started
   redis_1  | 1:C 17 Aug 22:11:10.480 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
   web_1    |  * Restarting with stat
   redis_1  | 1:M 17 Aug 22:11:10.483 * Running mode=standalone, port=6379.
   redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
   web_1    |  * Debugger is active!
   redis_1  | 1:M 17 Aug 22:11:10.483 # Server initialized
   redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
   web_1    |  * Debugger PIN: 330-787-903
   redis_1  | 1:M 17 Aug 22:11:10.483 * Ready to accept connections
   ```

   Compose 会拉取Redis镜像，然后为你的代码构建一个惊喜，并启动你定义的所有服务。在这个案例中，你的代码将会在构建时静态拷贝到镜像中。

2. 在浏览器中输入 http://localhost:5000 ，你将会看到你的应用正在运行。

   如果你是在你的本地使用Docker，那么你的web 应用正在你的Docker后台监听5000端口。在浏览器上输入 http://localhost:5000，将会看到`hello world`消息。如果看不到，你可以尝试输入`http://127.0.0.1:5000`。

   你将会看到

   ```shell
   Hello World! I have been seen 1 times.
   ```

   ![](https://docs.docker.com/compose/images/quick-hello-world-1.png)

3. 刷新页面。

   这个数字会增长

   ```shell
   Hello World! I have been seen 2 times.
   ```

![](https://docs.docker.com/compose/images/quick-hello-world-2.png)

4. 切换到其他终端窗口，键入`docker image ls`，将会列出本地镜像。列出的镜像中将会看到`redis`和`web`。

   ```shell
   $ docker image ls
   REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
   composetest_web         latest              e2c21aa48cc1        4 minutes ago       93.8MB
   python                  3.4-alpine          84e6077c7ab6        7 days ago          82.5MB
   redis                   alpine              9d8fa9aa0e5b        3 weeks ago         27.5MB
   ```

   你可以检查这个镜像，使用如下命令`docker inspect <tag or id>`。

5. 关于停止应用， 你可以在另一个终端中、你的项目目录下执行`docker-compose down`来终止； 或者也可以在之前执行`docker-compose up`的终端中键入 CTRL+C 来终止应用。



### 第五部：编辑Compose文件，增加数据卷绑定

编辑项目目录下的`docker-compose.yml`，为`web`服务增加一个绑定的数据卷。

```shell
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
  redis:
    image: "redis:alpine"
```

`volumes`关键字绑定了宿主机的项目目录和容器内的`/code`目录，这样就允许你快速的修改代码，快速生效，而不用重新构建镜像了。`environment`关键字设置了`FLASK_ENV`环境变量，它用来告诉`flask run`运行开发模式，并当代码变更时重新加载。这个模式只能用于开发。



### 第六步： 通过Compose重新构建和运行应用

在你的项目目录下，键入`docker-compose up`和更新了的Compose文件来构建你的应用，并运行应用。

```shell
$ docker-compose up
Creating network "composetest_default" with the default driver
Creating composetest_web_1 ...
Creating composetest_redis_1 ...
Creating composetest_web_1
Creating composetest_redis_1 ... done
Attaching to composetest_web_1, composetest_redis_1
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
...
```

检查`hello wolrd`是否出现在浏览器上，然后刷新浏览起，看数字是否增长。

> 共享的目录、数据卷 和挂载绑定
>
> 如果你的项目在用户目录（`cd ~`）之外，那么你需要共享出你使用的Dockerfile和数据卷的驱动或位置。如果你获得了一个运行时错误，该错误指出应用文件找不到、数据卷挂载被拒绝、或服务不能启动，请尝试使文件或驱动能被共享。数据卷挂载需要为用户目录（Windows： C:\Users;  Mac: /Users）之外的项目共享驱动，这对于使用Linux容器的Windows桌面Docker应用的项目是必须的。更多信息请参考see [Shared Drives](https://docs.docker.com/docker-for-windows/#shared-drives) on Docker Desktop for Windows, [File sharing](https://docs.docker.com/docker-for-mac/#file-sharing) on Docker for Mac。 一些常见的案例请参考 [Manage data in containers](https://docs.docker.com/engine/tutorials/dockervolumes/)。
>
> 如果你是在一个比较旧的Windows系统上使用VirtualBox，你可能会遇到一个关于共享目录的问题，它记录在了[VB trouble ticket](https://www.virtualbox.org/ticket/14920)。新的Windows系统上因为使用的是Docker桌面应用，而不再使用VirtualBox，所以符合条件。



### 第七步： 升级应用

由于我们应用代码是挂载到容器数据卷的，你可以修改代码并且立刻会看到变化，而不必重新构建镜像。

1. 修改代码`app.py`中的问候方式并保存。比如：改变`Hello World` 为`Hello from Docker`。

   ```shell
   return 'Hello from Docker! I have been seen {} times.\n'.format(count)
   ```

2. 在浏览器中刷新应用，问候语应该会改变，且计数仍然增长。

   ![](https://docs.docker.com/compose/images/quick-hello-world-3.png)

### 第八步： 体验一下其他的命令

如果你想在后台运行你的服务，你可以使用`-d`标签("detached"模式)来运行`docker-compose up`，然后用`docker-compose ps`来查看运行情况:

```shell
$ docker-compose up -d
Starting composetest_redis_1...
Starting composetest_web_1...

$ docker-compose ps
Name                 Command            State       Ports
-------------------------------------------------------------------
composetest_redis_1   /usr/local/bin/run         Up
composetest_web_1     /bin/sh -c python app.py   Up      5000->5000/tcp
```

`docker-compose run`命令允许你为你的服务执行一次性的命令。举个例子，查看`web`服务有哪些环境变量在生效中：

```shell
$ docker-compose run web env
```

`docker-compose --help`可以查看还有哪些可以使用的命令。你也可以为bash和zsh安装 [command completion](https://docs.docker.com/compose/completion/)，这样也可以做到还有哪些可用的命令。

如果你是通过 compose 命令`docker-compose up -d `启动的服务，你也可以通过下面的命令来停止服务：

```shell
$ docker-compose stop
```

你可以关掉一切，通过`down`命令，来删除所有的容器。加入`--volumns`还可以移除掉容器使用的数据卷

```shell
$ docker-compose down --volumes
```

这一节，你已经学会了Compose的基础。