### Service

#### 在K8S的世界里，虽然每个Pod都会被分配一个单独的IP地址，但这个IP地址会随着Pod的销毁而消失

#### Service（服务）就是用来解决这个问题的核心该你啊

#### 一个Service可以看作一组提供相同服务的Pod的对外访问接口

#### Service作用于哪些Pod是通过标签选择器来定义的



### Ingress

#### Ingress 是K8S机群里工作在OSI网络参考模型下，第七层的应用，对外暴露的接口

#### ==Service==只能进行L4流量调度，表现形式是 ip + port

#### ==Ingress==则可以调度不同的业务域、不同URL访问路径的业务流量

