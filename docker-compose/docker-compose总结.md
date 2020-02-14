#### docker-compose.yml 样例：

##### 各个标签的含义在注释里

```yaml
version: '3' # 选择的docker-compose 版本
services: # 要编排的一组服务
  fim-mysql: # 服务名称
    image: fim-mysql:1.0 # 镜像选择 后面的名称 为镜像及其版本，优先搜本地，后云端
    container_name: fim-mysql #根据镜像生成的容器的名字
    restart: always # 挂了后是否重启
    ports:
      - "3306:3306" # 暴露端口定义 宿主机:容器
    environment: # 环境变量定义
      MYSQL_ROOT_PASSWORD: 123456 #mysql镜像自带的一个属性，定义了它就是定义了root密码
      TZ: Asia/Shanghai # 选择时区

  fim-backend:
    image: fim-backend:1.0
    container_name: fim-backend
    restart: always
    ports:
      - "8080:8080"
    depends_on: # 依赖，在下面的镜像容器启动后再启动，如果容器有脚本执行，可能不会等脚本执行完毕
      - fim-mysql

  fim-frontend:
    image: fim-frontend:1.0
    container_name: fim-frontend
    restart: always
    ports:
      - "80:80"
    depends_on:
      - fim-mysql
      - fim-backend
```

#### 

#### docker-compose 执行命令，在docker-compose.yml所在目录执行命令

```shell
// 容器编排前台启动，会将几个容器的日志都实时地（一起）打印出来
docker-compose up
// 容器编排后台启动
docker-compose up -d
// 该容器编排的容器一起依次关闭，且删除之前产生的容器
docker-compose down
```



#### docker-compose安装

```shell
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
// 给权限
sudo chmod +x /usr/local/bin/docker-compose
// 测试
docker-compose --version
```





