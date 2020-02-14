##### 一个简单的 Dockerfile 样例

```dockerfile
#First Dockerfile
FROM unbuntu:14.04
MAINTAINER grady "xxx@xxx.com"
RUN apt-get update
RUN apt-get install -y nginx
EXPOSE 80
```



### 使用docker build  和Dockerfile 构建镜像

```shell
docker build [OPTIONS] PATH | URL | -
```

OPTIONS说明：

```shell
-f :指定要使用的Dockerfile路径；
-t: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签
```





---

#### docker 守护进程的管理

```shell
// 守护进程启动
service docker start
// 守护进程停止
service docker stop
// 守护进程重启
service docker restart
```



##### docker 启动配置文件

```shell
/etc/default/docker
```



---

#### Dockerfile 指令

##### EXPOSE

==暴露端口，即使在Dockerfile中写了，也需要在创建容器的指令中指定端口（这样设计时出于安全考虑）==



##### RUN

==在镜像构建时运行的命令==

---

##### CMD

```shell
CMD [ "executable", "param1", "param2" ] (exec 模式)
CMD command param1 param (shell 模式)
//参数模式
CMD ["param1", "param2"] (作为ENTRYPOINT指令的默认参数)
```



==在容器运行时执行的命令==

==如果docker run 有指定运行的命令，则CMD命令会被覆盖==

---

##### ENTRYPOINT

==不会被docker run所指定的命令覆盖==

==如果想覆盖，可使用docker run --entrypoint覆盖==

```shell
ENTRYPOINT [ "executable", "param1", "param2" ] (exec 模式)
ENTRYPOINT command param1 param2 (shell 模式)
```



---

##### ADD

```dockerfile
ADD 原路径 目标路径
```



==将本宿主机目录中的文件和目录拷贝到镜像中==

==目标路径必须写镜像中的绝对路径==

==ADD== 与==COPY== 的==区别==

```shell
1. ADD 包含tar的解压缩功能
2. 如果是单纯的复制文件，Docker推荐使用COPY
```



---

##### COPY

==如果是单纯的复制文件==

---

##### VOLUME

==提供数据持久化，共享数据的功能==

```dockerfile
VOLUME["/data"]
```



---

##### WORKDIR

==创建新容器时设置工作目录==

```dockerfile
WORKDIR /path/to/workdir
```



---

##### ENV

==设置环境变量==

```dockerfile
ENV key value
ENV key=value
```



---

##### USER

```dockerfile
USER xxx
```

==以哪种用户身份运行，例如***USER nginx**==

==如果不使用USER指令，默认使用root==



---

##### ONBUILD

==镜像触发器==

==当一个镜像被其他镜像作为基础镜像时执行==

例子：

```dockerfile
ONBUILD COPY index.html /usr/share/nginx/html/
//当作为父镜像被参与构建新镜像时，会执行上述回调指令
```



---

### Dockerfile 构建过程

##### 1. 从基础镜像中运行一个容器

##### 2. 执行一条指令，对容器作出修改

##### 3. 执行类似docker commit 的操作，提交一个新的镜像层

##### 4. 再基于刚提交的镜像运行一个新容器

##### 5. 执行Dockerfile中的下一条指令，直至所有指令执行完毕

