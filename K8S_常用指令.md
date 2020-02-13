### kubectl get 显示一个或更多resources资源

```shell
# 查看集群状态
kubectl get cs

# 查看集群节点信息
kubectl get nodes

# 查看集群命名空间
kubectl get ns

# 查看指定命名空间的服务
kubectl get svc -n kube-system

# 查看Pod详细信息
kubectl get pod <pod-name> -o wide

# 以yaml格式查看Pod信息
kubectl get pod <pod-name> -o yaml

# 查看资源对象，查看所有Pod列表
kubectl get pods

# 查看资源对象，查看rc和service列表
kubectl get rc, service

# 查看pod,svc,ep能及标签信息
kubectl get pod,svc,ep --show-labels 

# 查看所有的命名空间
kubectl get all --all-namespaces 
```



#### 查看集群信息

```shell
kubectl cluster-info
// 输出
Kubernetes master is running at https://172.xxx.xxx.145:6443
KubeDNS is running at https://172.xxx.xxx.145:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```



#### 在集群中运行一个指定镜像

```shell
kubectl run nginx --image=nginx --replicas=2 --port=80

// 输出
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx created
```



#### 暴露一个Deployment，生成一个Service

```shell
kubectl expose deployment nginx --port=80 --type=LoadBalancer
```



#### 查询服务

```shell
[root@k8s-master ~]# kubectl get service
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP        38h
nginx        LoadBalancer   10.96.94.236   <pending>     80:30800/TCP   20s

// 解释
1. 10.96.94.236 是 K8S 内部的IP
2. 80:30800 是 80是内部的端口；30800是暴露出来的端口
```



#### 描述一个服务（即查询详情）

```shell
kubectl describe service nginx
// 输出
Name:                     nginx
Namespace:                default
Labels:                   run=nginx
Annotations:              <none>
Selector:                 run=nginx
Type:                     LoadBalancer
IP:                       10.96.94.236
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30800/TCP
Endpoints:
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

