[Nacos Quick Start](https://nacos.io/zh-cn/docs/quick-start-docker.html)

### 首先我们肯定是Docker容器化安装

#### 1. 执行git clone 拉取代码

```shell
cd /usr/local/
mkdir docker
cd docker
git clone https://github.com/nacos-group/nacos-docker.git
cd nacos-docker
```

#### 2. 单机Mysql模式启动

```shell
docker-compose -f example/standalone-mysql.yaml up
```



### 这里遇到了两个坑

#### 1. 3306 端口号已被分配，导致启动失败，可以改动暴露的端口号，也可以关闭之前分配了3306端口的容器

#### 2.访问的url是需要带nacos的

```shell
http://172.16.5.132:8848/nacos
```

