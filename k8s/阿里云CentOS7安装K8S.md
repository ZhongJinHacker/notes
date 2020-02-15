### 1. 在阿里云山申请三台云服务器

##### 1.1 环境准备

##### 完成配置后的信息

| 服务器IP       | 操作系统 | CPU  | 内存 | 硬盘 | 主机名      | 节点角色 |
| -------------- | -------- | ---- | ---- | ---- | ----------- | -------- |
| 172.18.119.145 | centos7  | 2    | 4G   | 50G  | k8s-master  | master   |
| 172.18.119.150 | centos7  | 2    | 4G   | 50G  | k8s-node-01 | node     |
| 172.18.119.151 | centos7  | 2    | 4G   | 50G  | k8s-node-02 | node     |

##### 1.2 设置主机名称

```shell
#master
hostnamectl set-hostname k8s-master

#node01
hostnamectl set-hostname k8s-node01

#node02
hostnamectl set-hostname k8s-node02
```

##### 1.3 关闭防火墙

```shell
systemctl disable firewalled
systemctl stop firewalled
```

##### 1.4 关闭Selinux

```shell
sed  -i  "s/SELINUX=enforcing/SELINUX=disabled/g"  /etc/selinux/config
```

##### 1.5 关闭交换空间

```shell
swapoff -a
cp /etc/fstab /etc/fstab_bak
cat /etc/fstab_bak |grep -v swap > /etc/fstab
```

##### 1.6 配置 sysctl.conf

 ```shell
echo "net.bridge.bridge-nf-call-ip6tables = 1" >>/etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >>/etc/sysctl.conf
echo "net.ipv4.ip_forward = 1" >>/etc/sysctl.conf
echo "net.ipv4.ip_nonlocal_bind = 1" >>/etc/sysctl.conf
sysctl -p
 ```



### 2. 安装docker

##### 所有节点都必须安装docker

##### 2.1 配置aliyun的 docker yum源

```shell
yum install -y yum-utils 
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

##### 2.2 开始安装

```shell
yum install -y docker-ce-18.09.7 docker-ce-cli-18.09.7 
systemctl enable docker
systemctl start docker
docker -v
// 输出
Docker version 18.09.7, build 2d0083d
```



### 3. 安装Kubelet  Kubeadm  Kubectl

#####  所有节点都要安装 Kubelet 、Kubeadm、Kubectl

##### 3.1 配置 Kubernetes yum 源

```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```



##### 3.2 开始安装

```shell
yum remove -y kubelet kubeadm kubectl

// 阿里镜像当前是这个版本
yum install -y kebulet--1.17.2 kubeadm-1.17.2 kubectl-1.17.2
```



### 4. 部署K8S集群

#### 4.1 部署master节点

##### 4.1.1 配置kubeadm-config.yaml

```shell
cat <<EOF > /root/kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.17.2
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
controlPlaneEndpoint: "172.18.119.145:6443"
networking:
  serviceSubnet: "10.96.0.0/16"
  podSubnet: "10.100.0.0/20"
  dnsDomain: "cluster.local"
EOF
```

##### 4.1.2 拉取镜像

```shell
kubeadmin config images pull --config /root/kubeadm-config.yaml
```

##### 4.1.3 初始化Master, 并记录日志，里面的token很重要

```shell
kubeadm init --config=kubeadm-config.yaml |tee kubeadm_init.log
```

##### 4.1.4 配置Master认证权限

```shell
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

##### 4.1.5 安装 CNI 插件 Calico

```shell
cd  /root
rm  -f  calico.yaml
wget  https://docs.projectcalico.org/v3.8/manifests/calico.yaml
sed  -i 's/192.168.0.0/16/10.100.0.0/20/g'  calico.yaml
kubectl apply -f calico.yaml

# 这里是配置CALICO_IPV4POOL_CIDR，即Pods 所在的 IP 段
```



#### 4.2 部署 Node 节点

##### 4.2.1 拉取镜像

```shell
# 在master节点执行 复制 master 节点的 kubeadm-config.yaml 到所有 node 节点
rsync -avP /root/kubeadm-config.yaml 172.18.119.150:/root/
rsync -avP /root/kubeadm-config.yaml 172.18.119.151:/root/

# 在Node 节点执行，拉取镜像
kubeadmin config images pull --config /root/kubeadm-config.yaml
```

##### 4.3.2 初始化Node

##### 在子节点执行以下命令（从master节点里的 kubeadm_init.log中复制出最后一行）

 ```shell
kubeadm join 172.18.119.145:6443 --token vb2046.9zcbi5g1cxbwb0zx --discovery-token-ca-cert-hash sha256:da2f4b5c4ed8586c3ea7ad3a8054f0be6ccc328ceccc91aba3860b007394c666
 ```

##### 4.3.2 补充

##### 如果初始化master时创建的token已经过期，需要重新创建token和获取ca证书sha256编码hash值，然后再初始化node节点

```shell
# 先在 Master 节点执行
kubeadm token create --print-join-command
// 输出
kubeadm join 172.18.119.145:6443 --token xofceq.lqdfk5rt65tvgnqg --discovery-token-ca-cert-hash sha256:da2f4b5c4ed8586c3ea7ad3a8054f0be6ccc328ceccc91aba3860b007394c666

# 然后到Node节点中执行上述的输出
kubeadm join 172.18.119.145:6443 --token xofceq.lqdfk5rt65tvgnqg --discovery-token-ca-cert-hash sha256:da2f4b5c4ed8586c3ea7ad3a8054f0be6ccc328ceccc91aba3860b007394c666
```



#### 4.4 查看集群状态，当STATUS都为Ready时，集群搭建完成

```shell
kubectl get nodes
# 输出
NAME          STATUS   ROLES    AGE     VERSION
k8s-master    Ready    master   5d20h   v1.17.2
k8s-node-01   Ready    <none>   5d20h   v1.17.2
k8s-node-02   Ready    <none>   5d20h   v1.17.2
```



