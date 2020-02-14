##### Dockerfile  写法

```dockerfile
FROM nginx

MAINTAINER gradyjiang "jiangzhongjin@hotmail.com"

ENV LANG C.UTF-8

# 当前父目录
ENV PARENT_DIR .

COPY $PARENT_DIR/dist/ /usr/share/nginx/html/

COPY $PARENT_DIR/nginx.conf /etc/nginx/nginx.conf
```

##### 我将docker的内容独立到了一个目录中，也就是与==dist== 目录同一级了

#### 在这里我遇到了第一个坑

```shell
Dockerfile 默认只能在dockerfile所在目录工作，往上一级寻找是会报错的
所以就有一个需求，需要把dist目录拷贝到Dockerfile的目录下
```

构建脚本如下

```shell
echo "开始构建 fim-frontend 镜像..."

cp -rp ../dist ./fim-frontend

docker build -t fim-frontend:1.0 ./fim-frontend

rm -rf ./fim-frontend/dist
```



#### 利用ShellScript的能力就悄无声息地做到了

==有时，需要知识面全面，使用巧力==



#### PS：

#### nginx.conf 配置如下

```nginx
# grady config
worker_processes auto;
 
events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
 
    # log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
 
    # access_log  logs/access.log  main;
 
    sendfile        on;
 
    keepalive_timeout  65;

    client_max_body_size   20m;

    server {
      listen       80;
      charset utf-8;

      # access_log  logs/host.access.log  main;

      # 精确匹配/ 拿前端html
      location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
      }

      # 其他请求向后端转发
      location /chat {
        proxy_pass http://fim-backend:8080;
      }
 
      # redirect server error pages to the static page /50x.html
      #
      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }
    }

}
```



