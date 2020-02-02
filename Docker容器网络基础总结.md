### ifconfig 之 docker0



基于Linux的虚拟网桥（通用网络设备的抽象）

虚拟网桥特点：

##### 1. 可以设置IP地址

##### 2.相当于拥有一个隐藏的虚拟网卡



```shell
docker0 的地址划分
IP: 172.17.42.1 子网掩码: 255.255.0.0
MAC: 02:42:ac:11:00:00 到 02:42:ac:11:ff:ff
总共可提供65534个IP/MAC地址
```



---

#### 使用数据卷容器

```shell
docker run -it --volumns-from container ubuntu /bin/bash
```



==即使之后删除数据卷容器，使用了数据卷容器的容器依旧可以访问数据卷==