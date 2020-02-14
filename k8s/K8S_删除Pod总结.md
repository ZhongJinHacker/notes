#### K8S 不能直接删除Pod，直接删除Pod，会被Deployment重启



#### 删除前，必须先删除对应的Deployment

##### 例子：

```shell
// 查出Pod
[root@k8s-master ~]# kubectl get pods
NAME                     READY   STATUS             RESTARTS   AGE
nginx-65cbdd8bc4-mpkm4   0/1     ImagePullBackOff   0          37m
nginx-65cbdd8bc4-tbgt8   0/1     ImagePullBackOff   0          3h1m

// 查处其对应的deployment
[root@k8s-master ~]# kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/2     2            0           3h2m

// 删除对应的deployment
[root@k8s-master ~]# kubectl delete deployment nginx
deployment.apps "nginx" deleted

// 再次查询，发现deployment已被删除
[root@k8s-master ~]# kubectl get deployment
No resources found in default namespace.

// 再查看pod，Pod被成功删除
[root@k8s-master ~]# kubectl get pods
No resources found in default namespace.
```





