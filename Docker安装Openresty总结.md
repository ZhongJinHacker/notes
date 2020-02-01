#### 1. 启动Docker

```shell
systemctl start docker
```



#### 2. 查询有没有openresty镜像

```shell
docker search openresty -s 30
```

==-s 30 stars数大于30的==

#### 2. 拉取openresty镜像

```shell
docker pull openresty/openresty
```



#### 3. 执行镜像，生成容器

```shell
docker run  -it --name openresty -p 80:80 openresty/openresty /bin/bash
```



#### 4. 发现这个镜像里面没有 yum ，导致很不方便，干脆自己来



#### 5. 拉取centos，自己装

```shell
docker pull centos
```

#### 6. 执行容器

```shell
docker run  -it --name centos-web -p 80:80 centos /bin/bash
```



#### 7. 安装Openresty

##### 7.1 安装前准备，在你的 CentOS 系统中添加 `openresty` 仓库

```shell
yum install -y yum-utils
yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo

这个方法不行
```

##### 7.2 安装

```shell
yum install -y openresty
这个方法不行
```

##### 7.3 安装报错，换一个方式（源码包编译安装）装

```shell
yum install -y wget
wget http://openresty.org/download/openresty-1.15.8.2.tar.gz

安装一些要用到的开发库
yum install pcre-devel openssl-devel gcc curl -y

tar -zxvf openresty-1.15.8.2.tar.gz

安装perl
yum install perl*

配置
./configure --prefix=/opt/openresty --with-luajit --without-http_redis2_module --with-http_iconv_module

编译并安装
make
make install 
```

##### 7.4 启动 Openresty

```shell
./nginx/sbin/nginx
```



##### 7.5 查看运行docker 的机器的ip

浏览器访问 http://ip:80/ 正常