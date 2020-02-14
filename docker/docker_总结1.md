

#### docker -v /hostDir:/containerDir

==/hostDir==为宿主机的目录

==/containerDir==为容器内的目录

**-v 实现两个目录的挂在，即容器内数据持久化到本机**

---

#### docker ps

**参数：**

​	-a:  显示所有容器(包括没有在运行的容器)

​	-q: 仅显示容器ID

**查看正在运行的容器**

---

#### docker images 或 docker image ls

**查看本地有哪些镜像**

---

#### docker rmi `imageID或镜像名`

**删除镜像, 加`-f` 强制删除镜像（有生成容器的镜像需要用`-f`）**

---

#### docker container ls  -a

**显示所有容器（包括没有在运行的)**

---

#### docker rm `containerID`

**删除容器**

---

#### docker run  `imageID或镜像名`

**执行镜像生成运行的容器**

---

#### docker search -f=stars=1000 java

`-f`使用过滤器

`--limit 2`最多显示2条

**寻找叫java的镜像，同时stars数超过1000的**

---

#### docker run -d -p 91:80 nginx

**表示 后台运行Nginx 映射宿主机91端口到容器的80端口**

##### 常用参数选项：

​	-d  => 后台运行

​	-P  => 随机端口映射

​	-p => 指定端口映射 例子：`-p 91:80` ，前为宿主机端口，后为容器端口

---

#### 进入容器

#### docker attach `conatainerID`

---



