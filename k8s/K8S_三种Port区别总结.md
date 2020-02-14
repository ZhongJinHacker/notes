#### nodePort:
外部流量访问K8S集群中Service入口的一种方式
	
比如外部用户要访问k8s集群中的一个Web应用，那么我们可以配置对应service的type=NodePort，nodePort=30001。其他用户就可以通过浏览器http://node:30001访问到该web服务。

---



#### port:

K8S集群内部服务之间访问service的入口。即clusterIP:port是service暴露在clusterIP上的端口

---



#### targetPort:

#### 容器的端口（最终的流量端口）。targetPort是“pod”上的端口，从port和nodePort上来的流量，经过kube-proxy流入到后端pod的targetPort上，最后进入容器。



#### 例子：

```yaml
apiVersion: v1
kind: Service
metadata:
 name: nginx-service
spec:
 type: NodePort         // 有配置NodePort，外部流量可访问k8s中的服务
 ports:

- port: 30080        // 服务访问端口
targetPort: 80       // 容器端口
nodePort: 30001      // NodePort
 selector:
  name: nginx-pod

```



  

### 总结：
总的来说，port和nodePort都是service的端口，前者暴露给k8s集群内部服务访问，后者暴露给k8s集群外部流量访问。从上两个端口过来的数据都需要经过反向代理kube-proxy，流入后端pod的targetPort上，最后到达pod内的容器。