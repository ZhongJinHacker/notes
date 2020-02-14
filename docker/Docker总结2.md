##### 列出本机包含的镜像

```shell
docker images [OPTIONS] [REPOSITORY]
```

OPTIONS说明：

==-a 列出本地所有的镜像==

==-f 显示满足条件的镜像==

==-q 只显示镜像ID==



##### 查看镜像的详细信息

```shell
docker inspect [OPTIONS] CONTAINER|IMAGE [CONTAINER|IMAGE...]
```

OPTIONS说明

==-f 指定返回值的模版文件==

==-s 显示总的文件大小==



##### 删除镜像

```shell
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

==-f 强制删除==

==--no-prune :不移除该镜像的过程镜像，默认移除==



##### 查找镜像

```shell
docker search [OPTION] TERM
```

==--automated 只列出automated build类型的镜像==

==--no-trunc 显示完整的镜像描述==

==-s 列出收藏数不小于指定值的镜像==

##### 使用search命令一次最多返回25个结果





##### 拉取镜像

```shell
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

OPTIONS说明:

==-a 拉取所有tagged镜像==

==--disable-content-trust 忽略镜像的校验，默认开启==



##### 如何配置国内镜像仓库

```shell
1. 打开/etc/default/docker
2. 添加 DOCKER_OPTS = "--registry-mirror=http://MIRROR-Addr"
```



##### 推送镜像

```shell
docker push [OPTIONS] NAME[:TAG]
```

OPTIONS说明：

==--disable-content-trust 忽略镜像校验，默认开启==



#### 构建镜像

```shell
1. 保存对容器的修改，并在此使用
2. 自定义镜像的呢你
3. 以软件的形式打包，并分发服务及其运行环境
```



##### 通过容器构建镜像

```shell
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

OPTIONS说明：

==-a 提交的镜像作者名字==

==-c 使用Dockerfile指令来创建镜像==

==-m 提交的说明文字信息==

==-p 在commit时，将容器暂停==



##### 通过Dockerfile 文件构建镜像

```shell
docker build
```











