### K8S 核心组件

1. 配置存储中心 --> etcd服务
2. 主控（master）节点

​         [1] kube-apiserver 服务

```shell
apiserver:（K8S 大脑）
1. 提供了集群管理的 REST API 接口（包括 鉴权、数据校验及集群状态变更）
2. 负责其他模块之间的数据交互，承担通信枢纽功能
3. 是治愈啊配额控制的入口
4. 提供完备的集群安全机制
```

​         [2] kube-controller-manager服务

```shell
kube-controller-manager:(控制器管理器)
1. 由一系列控制器组成，通过apiserver监控整个集群的状态，并确保集群处于预期的工作状态
比如一下控制器：
1. Node Controller
2. Deployment Controller
3. Service Controller
4. Volume Controller
5. Endpoint Controller
6. Garbage Controller
7. Namespace Controller
8. Job Controller
9. Resource quta Controller
...
```

​         [3] kube-schedule服务

```shell
schedule:
1. 接收调度pod到适合的运算节点上
2. 预算策略（predict）
3. 优选策略（priorities）
```



3. 运算（node）节点

      [1] kube-kubelet 服务

   ```shell
   kubelet:
   1. 主要功能是定时从某个地方获取节点上pod的期望状态（运行什么容器、运行的副本数量、网络或者存储如何配置等等），并调用对应的容器平台接口到达这个状态
   2. 定时汇报当前节点的状态给apiserver，以供调度的时候使用
   3. 镜像和容器的清理工作，保证节点上镜像不会占满磁盘空间，退出的容器不会占用太多资源
   ```

   

      [2] kube-proxy 服务

```shell
kube-proxy:
1. 是K8S在每个节点上运行网络的代理，service资源的载体
2. 建立了Pod网络和集群网络的关系（slusterIp -> podIp)
3. 常用的流量调度模式：[1]Userspace(废弃), [2]Iptables(濒临废弃), [3]Ipvs(推荐)
4. 负责建立和删除包括更新调度规则、通知apiserver自己的更新，或者从apiserver那里获取其他kube-proxy的调度规则来更新自己
```



### K8S Cli客户端

      ##### ==kubectl==



#### 核心插件

1. CNI 网络插件 --> flannel/calico
2. 服务发现用插件 --> coredns
3. 服务暴露用插件 --> traefik
4. GUI管理插件 --> Dashboard





