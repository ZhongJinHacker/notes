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

