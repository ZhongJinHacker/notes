启动容器（在新容器中执行命令）：

```shell
docker run IMAGE [COMMAND][ARG...]
```



交互方式启动新容器：

```shell
docker run -i -t IMAGE /bin/bash
```

==-i 表示 --interactive=true  默认是false==

==-t  表示 --tty=true ，即为容器分配一个伪tty终端  默认为false==



查看容器

```shell
docker ps [-a] [-l]
```

==-a 表示列出所有容器==

==-l列出最新创建出来的容器==



查看容器详情

```shell
docker inspect container-id
```



自定义容器名：

```shell
docker run --name=自定义名 -i -t IMAGE /bin/bash
```



##### 重新启动已经停止的容器

```shell
docker start [-i] 容器名
```

==-i 交互式打开==



删除已停止的容器

```shell
docker rm 容器名/container-id
```



---

### 守护式容器

1.即能够长期运行

2.没有交互式会话

3.适合运行应用程序和服务

---

以守护形式运行容器

```shell
方法1:
docker run -i -t IMAGE /bin/bash
Ctrl+P Ctrl+Q

方法2:
docker run -d IMAGE [COMMAND][ARG...]
-d 表示后台运行;COMMAND 执行完容器也会停止
```



查看容器日志：

```shell
docker logs [-f][-t][--tail] 容器名
```

==-f 表示--follows=true 表示追踪并返回结果  默认false==

==-t 表示--timestamps=true 把时间打印出来  默认false==

==--tail 表示返回指定数量的结尾日志，不指定则返回所有日志== 



查看容器内的进程

```shell
docker top 容器名/container-id
```



#### 在运行的容器内启动新进程

```shell
docker exec [-d][-i][-t] 容器名/Container-id [COMMAND][ARG...]
```



停止守护式容器：

```shell
docker stop 容器名/container-id
docker kill 容器名/container-id
```

stop 会正常停止；kill会直接停止容器



```shell
Ctrl+P 组合 Ctrl+Q
```

将交互式容器放到后台运行



```shell
docker run -d
```

以后台运行的方式运行一个容器（后面的命令执行完毕 则容器也会停止）





---

查询本机docker镜像

```shell
[root@localhost ~]# docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
openresty/openresty   latest              3c9e8cc37fa4        3 days ago          84.8MB
```

