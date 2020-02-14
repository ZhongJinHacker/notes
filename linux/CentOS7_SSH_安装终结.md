#### 在使用ssh 连接自己的centos 虚拟机时，发现连接不上，于是有了这个安装过程

(以下是在root用户下执行的，没权限的话就sudo)

##### 1.首先判断是否有这个服务

```shell
systemctl list-unit-files | grep sshd
```

发现没有

##### 2. 判断是否有用yum 安装了这个服务

```shell
yum list installed | grep openssh-server
```

发现有；如果没有执行安装命令`yum install openssh-server`

##### 3. 到`/etc/ssh/`下找到并打开`sshd_config`文件

```shell
解开以下注释
Port 22

ListenAddress 0.0.0.0
ListenAddress ::

PermitRootLogin yes

PasswordAuthentication yes
```



##### 4. 开启sshd服务

```shell
service sshd start
```

查看进程检查是否开启成功

```shell
ps aux |grep sshd
```

或者查看端口22 是否被监听

```shell
netstat -an | grep 22

显示：
tcp 0 0.0.0.0:22 0.0.0.0:* LISTEN
```



##### 5. 将sshd 服务添加到自启动列表中

```shell
systemctl enable sshd
```



