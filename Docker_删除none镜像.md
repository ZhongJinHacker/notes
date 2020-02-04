#### Docker 删除 none 镜像

```shell
docker images|grep none|awk '{print $3}'|xargs docker rmi
```

