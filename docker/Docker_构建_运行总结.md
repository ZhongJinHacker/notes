### 样例：

#### 构建镜像

#### build-image-fim-backend.sh

```shell
echo "开始构建 fim-backend 镜像..."

cp -rp ../target/fim-backend-*.jar ./fim-backend/

docker build -t fim-backend:1.0 ./fim-backend

rm -rf  ./fim-backend/fim-backend-*.jar
```

==-t 给镜像命名和TAG==

==结尾处路径表示，工作目录，即Dockerfile所在位置==



#### 运行容器

#### docker-run-fim-backend.sh

```shell
docker run --name fim-backend -p 8080:8080  -d fim-backend:1.0
```

==--name 为容器命名==

==-p 暴露端口映射 宿主机端口:容器端口==

==-d 后台运行==

==最后结尾处是选择的镜像名与TAG==

