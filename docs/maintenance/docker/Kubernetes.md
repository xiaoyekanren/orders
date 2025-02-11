# Kubernetes
## 安装
 - 记录安装日期：20240924   
 - 版本：1.31  
 - 使用containerd   
 - 基于 ubuntu22.04.5   

### 前置
1. 关闭swap  
2. 配置hostname & 免密   
3. 配置时间同步   
4. 加载内核模块   
 - overlay: 这是用于 OverlayFS 的模块，Kubernetes 使用它来管理容器文件系统。    
 - br_netfilter: 这是用于支持网络过滤的模块，Kubernetes 需要它来处理网络流量。  
``` shell
# vim /etc/modules-load.d/k8s.conf 
# 或者直接写到/etc/modules
overlay
br_netfilter
# 临时生效
modprobe overlay
modprobe br_netfilter
# 验证
lsmod | grep -e "overlay" -e "br_netfilter"
```

5. 开启转发  
 - net.bridge.bridge-nf-call-iptables=1: 允许 iptables 处理通过桥接网络接口的流量。  
 - net.bridge.bridge-nf-call-ip6tables=1: 允许 ip6tables 处理通过桥接网络接口的 IPv6 流量。  
 - net.ipv4.ip_forward=1: 允许 IPv4 数据包转发。  
``` shell
# vim /etc/sysctl.d/k8s.conf
# 或者直接写到/etc/sysctl.conf, 追加到末尾

# k8s
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1

# 执行sysctl -p重新加载
```

6. 安装ipset和ipvsadm
``` shell
apt-get install ipset ipvsadm -y

# 加载模块
# vim /etc/modules-load.d/ipvs.conf
# 或者直接写到/etc/modules
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack

# 执行以下临时生效
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack

# 验证
lsmod | grep -e "ip_vs" -e "nf_conntrack"
```

7. 安装其他依赖
``` shell
# socat，kubeadm join时要用
apt install socat
```

### containerd 安装
1. 安装
2. 生成配置文件
``` shell
containerd config default > /etc/containerd/config.toml
```

3. 修改配置参数
```  shell
# vim /etc/containerd/config.toml
# sandbox_image = "registry.k8s.io/pause:3.8"
sandbox_image = "registry.k8s.io/pause:3.9"
# 或者改国内源，sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
# pause 镜像是一个非常小的镜像，通常用于 Kubernetes 中作为 Pod 的基础，即根容器
# 如果安装了k8s v1.3，这里改成pause:3.10

# SystemdCgroup = false
SystemdCgroup = true
```

### kubectl & kubeadm & kubelet 安装
#### 安装
使用阿里云的文档来安装最新版，文档里只写到了1.29，可通过浏览[地址](https://mirrors.aliyun.com/kubernetes-new/core/stable/)，确定最新版本  

文档：https://developer.aliyun.com/mirror/kubernetes  
``` shell
apt-get update && apt-get install -y apt-transport-https

curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.31/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.31/deb/ /" | tee /etc/apt/sources.list.d/kubernetes.list

apt-get update && apt-get install -y kubelet kubeadm kubectl
```

#### 配置开机自启
``` shell
# 1. 拷贝配置文件
cp /etc/default/kubelet  /etc/sysconfig/

# 2. 增加参数
# vim /etc/sysconfig/kubelet
# KUBELET_EXTRA_ARGS=
KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"

# 3. 开机自启
systemctl enable kubelet
```

#### 禁止apt更新
``` shell
# 禁止
apt-mark hold kubelet kubeadm kubectl

# 取消禁止
apt-mark unhold kubelet kubeadm kubectl
```

#### 预下载镜像
``` shell
# 显示
kubeadm config images list
# pull
kubeadm config images pull

## 阿里镜像
kubeadm config images list --image-repository registry.aliyuncs.com/google_containers
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers

# 配置crictl
# vim /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
#  执行crictl images 查看拉取的镜像

```

### 集群配置  
#### 配置文件  
注意，以下未说明，都是在主节点执行。  
1. 生成配置文件  
``` shell
kubeadm config print init-defaults > init.yaml
```

2. 修改配置文件  
``` shell
localAPIEndpoint:
# advertiseAddress: 1.2.3.4
advertiseAddress: 172.16.31.86  # master节点的ip

nodeRegistration:
# name: node
name: a86  # 当前节点的标识

# imageRepository: registry.k8s.io
imageRepository: registry.aliyuncs.com/google_containers

networking: 
# serviceSubnet: 10.96.0.0/12
serviceSubnet: 10.10.0.0/16
# 新增一行，后续calico用
podSubnet: 10.20.0.0/16
```

3. 初始化集群  
``` shell
kubeadm init --config init.yaml 
```
注意安装完成之后的信息！
``` shell 
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as ro

kubeadm join 172.20.31.86:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:45ce1bdeeb264124fc1b02193e86f37bbdc4
```
里面包含4个事。
``` shell
# 1. 设置环境变量，注意普通用户和root用户不一致
# 2. deploy a pod network
# 3. 加入集群的方式
```

4. 拷贝集群配置文件 & 新增环境变量  
``` shell
# 拷贝配置文件(普通用户)
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 新增环境变量(root用户)
# vim ~/.bashrc
export KUBECONFIG=/etc/kubernetes/admin.conf
source ~/.bashrc
```

5. 加入其他节点  
**注意：只有这一步是要在其他节点执行的，其他全部在主节点执行。**  
``` shell
kubeadm join 172.20.31.86:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:xxxxxx
```
通过以下命令查看集群状态：  
``` shell
# 查看节点状态
# 如下所示， Status是NotReady
# root@a86:/data# kubectl get nodes
NAME   STATUS     ROLES           AGE     VERSION
a86    NotReady   control-plane   14m     v1.31.1
a87    NotReady   <none>          3m10s   v1.31.1
a88    NotReady   <none>          3m      v1.31.1

# 查看pods，最上面2个coredns是需要调度的pod，因为节点状态时NotReady，所以这里是Pending
# root@a86:/data# kubectl get pods -n kube-system
NAME                          READY   STATUS    RESTARTS   AGE
coredns-7c65d6cfc9-kqsjf      0/1     Pending   0          13m
coredns-7c65d6cfc9-vnzln      0/1     Pending   0          13m
etcd-a86                      1/1     Running   0          13m
kube-apiserver-a86            1/1     Running   0          13m
kube-controller-manager-a86   1/1     Running   0          13m
kube-proxy-lqg2h              1/1     Running   0          13m
kube-proxy-s7k2j              1/1     Running   0          2m33s
kube-proxy-s9svg              1/1     Running   0          2m43s
kube-scheduler-a86            1/1     Running   0          13m
```

### calico安装
即：deploy a pod network to the cluster  
文档： https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart   
版本：3.28   

1. 下载yaml  
``` shell
# tigera-operator
wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/tigera-operator.yaml
# custom-resources
wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/custom-resources.yaml
``` 

2. 部署tigera-operator  
``` shell
kubectl create -f tigera-operator.yaml

# 查看命名空间
# kubectl get ns
# root@a86:~/documents# kubectl get ns
NAME              STATUS   AGE
default           Active   24m
kube-node-lease   Active   24m
kube-public       Active   24m
kube-system       Active   24m
tigera-operator   Active   28s  # 新增

# 查看pods
# root@a86:~/documents# kubectl get pods -n tigera-operator
NAME                              READY   STATUS    RESTARTS   AGE
tigera-operator-89c775547-mm4zd   1/1     Running   0          84s
``` 

3. 部署custom-resources    
修改yaml文件的ip段，和init.yaml的podSubnet保持一致  
``` shell 
# vim custom-resources.yaml
# cidr: 192.168.0.0/16
cidr: 10.20.0.0/16

# create
kubectl create -f custom-resources.yaml 

# 查看命名空间
# root@a86:~/documents/calico# kubectl get ns
NAME              STATUS   AGE
calico-system     Active   49s  # 新增
default           Active   34m
kube-node-lease   Active   34m
kube-public       Active   34m
kube-system       Active   34m
tigera-operator   Active   10m

# 查看pods
kubectl get pods -n calico-system
root@a86:~/documents/calico# kubectl get pods -n calico-system
NAME                                       READY   STATUS              RESTARTS   AGE
calico-kube-controllers-7c697cc6d9-ld96r   0/1     ContainerCreating   0          107s
calico-node-7fmck                          0/1     Init:1/2            0          107s
calico-node-kb9n9                          0/1     PodInitializing     0          107s
calico-node-n9n7v                          0/1     Init:1/2            0          107s
calico-typha-78d85b78db-jdtwd              1/1     Running             0          104s
calico-typha-78d85b78db-n7c5q              1/1     Running             0          107s
csi-node-driver-slj67                      0/2     ContainerCreating   0          107s
csi-node-driver-wqqrm                      0/2     ContainerCreating   0          107s
csi-node-driver-wwv8s                      0/2     ContainerCreating   0          107s
``` 
4. 漫长等待......  

### 尾声
1. 查看集群相关的状态  
如下所示，全部成功运行
``` shell 
# root@a86:~/documents/calico# kubectl get pods -n calico-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-7c697cc6d9-ld96r   1/1     Running   0          4m17s
calico-node-7fmck                          1/1     Running   0          4m17s
calico-node-kb9n9                          1/1     Running   0          4m17s
calico-node-n9n7v                          1/1     Running   0          4m17s
calico-typha-78d85b78db-jdtwd              1/1     Running   0          4m14s
calico-typha-78d85b78db-n7c5q              1/1     Running   0          4m17s
csi-node-driver-slj67                      2/2     Running   0          4m17s
csi-node-driver-wqqrm                      2/2     Running   0          4m17s
csi-node-driver-wwv8s                      2/2     Running   0          4m17s

# root@a86:~/documents/calico# kubectl get pods -n kube-system
NAME                          READY   STATUS    RESTARTS   AGE
coredns-7c65d6cfc9-kqsjf      1/1     Running   0          37m
coredns-7c65d6cfc9-vnzln      1/1     Running   0          37m
etcd-a86                      1/1     Running   0          38m
kube-apiserver-a86            1/1     Running   0          38m
kube-controller-manager-a86   1/1     Running   0          38m
kube-proxy-lqg2h              1/1     Running   0          37m
kube-proxy-s7k2j              1/1     Running   0          27m
kube-proxy-s9svg              1/1     Running   0          27m
kube-scheduler-a86            1/1     Running   0          38m

# root@a86:~/documents/calico# kubectl get nodes
NAME   STATUS   ROLES           AGE   VERSION
a86    Ready    control-plane   38m   v1.31.1
a87    Ready    <none>          27m   v1.31.1
a88    Ready    <none>          27m   v1.31.1

# root@a86:~/documents/calico# kubectl get pods -n kube-system -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP             NODE   NOMINATED NODE   READINESS GATES
coredns-7c65d6cfc9-kqsjf      1/1     Running   0          40m   10.20.43.65    a86    <none>           <none>
coredns-7c65d6cfc9-vnzln      1/1     Running   0          40m   10.20.43.66    a86    <none>           <none>
etcd-a86                      1/1     Running   0          40m   172.20.31.86   a86    <none>           <none>
kube-apiserver-a86            1/1     Running   0          40m   172.20.31.86   a86    <none>           <none>
kube-controller-manager-a86   1/1     Running   0          40m   172.20.31.86   a86    <none>           <none>
kube-proxy-lqg2h              1/1     Running   0          40m   172.20.31.86   a86    <none>           <none>
kube-proxy-s7k2j              1/1     Running   0          29m   172.20.31.88   a88    <none>           <none>
kube-proxy-s9svg              1/1     Running   0          30m   172.20.31.87   a87    <none>           <none>
kube-scheduler-a86            1/1     Running   0          40m   172.20.31.86   a86    <none>           <none>
```

