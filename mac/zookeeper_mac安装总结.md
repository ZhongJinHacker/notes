### Zookeeper  mac安装总结

#### 1. 执行

```shell
brew install zookeeper
```

可能遇到报错

```shell
Error: The following directories are not writable by your user:
/usr/local/share/man/man5

You should change the ownership of these directories to your user.
  sudo chown -R $(whoami) /usr/local/share/man/man5

And make sure that your user has write permission.
  chmod u+w /usr/local/share/man/man5
```

这时需要执行以下命令修正错误

```shell
➜  ~ sudo chown -R `whoami`:admin /usr/local/bin
➜  ~ sudo chown -R `whoami`:admin /usr/local/share
```

然后再执行

```shell
brew install zookeeper
```

安装成功后，zookeeper的配置目录在**/usr/local/etc/zookeeper**

```shell
➜  ~ ll /usr/local/etc/zookeeper
total 32
-rw-r--r--  1 Jiangzhongjin  admin  346 Aug 18 16:40 log4j.properties
drwxr-xr-x  6 Jiangzhongjin  admin  192 Aug 18 16:40 .
-rw-r--r--  1 Jiangzhongjin  admin  941 Aug 18 16:40 zoo_sample.cfg
-rw-r--r--  1 Jiangzhongjin  admin  941 Aug 18 16:40 zoo.cfg
-rw-r--r--  1 Jiangzhongjin  admin   67 Aug 18 16:40 defaults
drwxr-xr-x  9 Jiangzhongjin  admin  288 Aug 18 16:40 ..
```



#### 2.  如果需要修改配置，直接修改/usr/local/etc/zookeeper/zoo.cfg



### 3. 执行下面命令启动

 ```shell
zkServer start
 ```



#### 4. zookeeper 其他命令总结

```shell
zkServer start //启动zookeeper
zkServer stop //停止（关闭）zookeeper
zkServer status //查看zookeeper状态

zkCli //连接本地zookeeper，可接IP:port连接远端zookeeper
```

