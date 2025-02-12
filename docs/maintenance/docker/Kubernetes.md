# Kubernetes
## 1. 安装
 - 记录安装日期：2024-09-24   
 - 版本：1.31  
 - 使用containerd   
 - 基于 ubuntu22.04.5   
 - 修改于： 2025-02-11

### 1.1 前置

#### 1.1.1 关闭swap  
无
#### 1.1.2 配置hostname & 免密   
无
#### 1.1.3 配置时间同步   
无
#### 1.1.4 加载内核模块   
 - overlay: 这是用于 OverlayFS 的模块，Kubernetes 使用它来管理容器文件系统。    
 - br_netfilter: 这是用于支持网络过滤的模块，Kubernetes 需要它来处理网络流量。  

修改文件`/etc/modules`，新增：  
``` shell
# k8s
overlay
br_netfilter
```

使模块临时生效：  
``` shell
modprobe overlay
modprobe br_netfilter
```

验证是否生效：  
`lsmod | grep -e "overlay" -e "br_netfilter"`  

#### 1.1.5 开启转发  
修改文件`/etc/sysctl.conf`, 追加到末尾  

``` shell
# k8s
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
```

执行`sysctl -p`重新加载。  

解释：  
 - net.bridge.bridge-nf-call-iptables=1: 允许 iptables 处理通过桥接网络接口的流量。  
 - net.bridge.bridge-nf-call-ip6tables=1: 允许 ip6tables 处理通过桥接网络接口的 IPv6 流量。  
 - net.ipv4.ip_forward=1: 允许 IPv4 数据包转发。  


#### 1.1.6 安装ipset和ipvsadm
命令：`apt-get install ipset ipvsadm -y`

修改文件`/etc/modules`，新增：  
``` shell
# k8s
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
```

使模块临时生效：  
``` shell
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
```

使用命令`lsmod | grep -e "ip_vs" -e "nf_conntrack"`验证。  


### 1.2 安装containerd  

#### 1.2.1 安装  
无
#### 1.2.2 生成配置文件  
使用命令：   
`containerd config default > /etc/containerd/config.toml`   

#### 1.2.3 修改配置参数
编辑文件`/etc/containerd/config.toml`：
```  shell
# sandbox_image = "registry.k8s.io/pause:3.8"
sandbox_image = "registry.k8s.io/pause:3.9"
# 或者改国内源，sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"

# SystemdCgroup = false
SystemdCgroup = true
```

pause 镜像是一个非常小的镜像，通常用于 Kubernetes 中作为 Pod 的基础，即根容器。  

> 2025-02-11：如果安装了k8s v1.32，这里改成**pause:3.10**，不然后面会有 warn。  

#### 1.2.4 配置代理
kubernet拉取镜像，也是用的containerd的代理。  
所以如果网络不好，注意配置containerd的代理。  

### 1.3 安装 k8s (kubectl & kubeadm & kubelet) 

#### 1.3.1 安装
使用阿里云的[文档](https://developer.aliyun.com/mirror/kubernetes)来安装最新版，文档里只写到了1.29，实际可通过查看[地址](https://mirrors.aliyun.com/kubernetes-new/core/stable/)，确定最新版本。  

命令如下，**注意修改版本**：  

``` shell
apt-get update && apt-get install -y apt-transport-https

curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.31/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.31/deb/ /" | tee /etc/apt/sources.list.d/kubernetes.list

apt-get update && apt-get install -y kubelet kubeadm kubectl
```

#### 1.3.2 禁止apt更新

``` shell
# 禁止
apt-mark hold kubelet kubeadm kubectl

# 取消禁止
apt-mark unhold kubelet kubeadm kubectl
```

#### 1.3.3 配置开机自启  
1. 拷贝配置文件  
`cp /etc/default/kubelet  /etc/sysconfig/`  

2. 修改文件`/etc/sysconfig/kubelet`  
``` shell
KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"  # 就一行
```

3. 开机自启  
`systemctl enable kubelet`  

#### 1.3.4 预下载镜像
``` shell
# 显示
kubeadm config images list
# pull
kubeadm config images pull

```
网络不好的话，使用阿里镜像查看和下载。   
``` shell
# 使用阿里镜像
kubeadm config images list --image-repository registry.aliyuncs.com/google_containers
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers
```

可使用 `crictl images` 查看拉取的镜像。该命令执行时会有报警，通过编辑/创建文件 `/etc/crictl.yaml`来避免，文件内容如下：
``` shell
# /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
```


### 1.4 集群配置  
注意，以下未说明，都是在主节点执行。  

#### 1.4.1 初始化集群  
1. 生成配置文件（使用后可丢弃）  
`kubeadm config print init-defaults > init.yaml`   

2. 修改配置文件 `init.yaml`

``` shell
localAPIEndpoint:
# advertiseAddress: 1.2.3.4
advertiseAddress: 172.16.31.86  # 修改为主节点的ip

nodeRegistration:
# name: node
name: a86  # 主节点的标识

# imageRepository: registry.k8s.io
imageRepository: registry.aliyuncs.com/google_containers  # 切换到阿里云的镜像仓库，但是好像没用

networking: 
# serviceSubnet: 10.96.0.0/12
serviceSubnet: 10.10.0.0/16
podSubnet: 10.20.0.0/16  # 新增，后续calico用
```

3. 初始化集群  
`kubeadm init --config init.yaml`  

> 注意安装完成之后的信息：

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
里面包含3个事：  
  1. 设置环境变量，注意普通用户和root用户不一致  
  2. deploy a pod network，例如calico  
  3. 加入集群的命令，注意保存  

##### 1.4.1.1 设置环境变量
1. 普通用户（拷贝配置文件）
``` shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

2. root用户（新增环境变量）
``` shell
# vim ~/.bashrc
export KUBECONFIG=/etc/kubernetes/admin.conf
# source ~/.bashrc
```

##### 1.4.1.2 calico安装
即：**deploy a pod network to the cluster** ，[参考文档](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart) 

1. 下载2个yaml文件  
``` shell
# tigera-operator
wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/tigera-operator.yaml
# custom-resources
wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/custom-resources.yaml
``` 

2. 创建 tigera-operator  
`kubectl create -f tigera-operator.yaml`  

3. 创建 custom-resources  

修改yaml里的`cidr`，和文件`init.yaml`的`podSubnet`保持一致。

``` shell
# custom-resources.yaml
cidr: 10.20.0.0/16
```

执行create命令  
`kubectl create -f custom-resources.yaml`

##### 1.4.1.3 加入其他节点  
> **注意：只有这一步是要在其他节点执行的，其他全部在主节点执行。**    

安装socat，否则可能会报错。  
`apt-get install socat`  

``` shell
kubeadm join 172.20.31.86:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:xxxxxx
```

通过以下命令查看集群状态：  
当前状态就是NotReady，得后续操作完成之后才会变化。  

``` shell
# 查看节点状态
# 如下所示， Status是NotReady
# kubectl get nodes
NAME   STATUS     ROLES           AGE     VERSION
a86    NotReady   control-plane   14m     v1.31.1
a87    NotReady   <none>          3m10s   v1.31.1
a88    NotReady   <none>          3m      v1.31.1
```

#### 1.4.2 漫长等待......  

执行如下命令查看状态，直到 pods 全部为 Running，node 全为 Running。  

```shell
kubectl get pods -n calico-system
kubectl get pods -n kube-system
kubectl get nodes
```
> 注：kube-system 的 dnscore 依赖 calico。

如下所示，说明安装完成。
``` shell
# kubectl get pods -n kube-system
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

# kubectl get pods -n calico-system
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

# kubectl get nodes
NAME   STATUS   ROLES           AGE   VERSION
a86    Ready    control-plane   38m   v1.31.1
a87    Ready    <none>          27m   v1.31.1
a88    Ready    <none>          27m   v1.31.1
```
### 1.5 插件
[文档在此。](https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/web-ui-dashboard/)
#### 1.5.1 安装 kubernetes-dashboard
1. 安装helm
参考[文档](https://helm.sh/zh/docs/intro/install/)。  
2. 官方命令
```shell
# 添加 kubernetes-dashboard 仓库
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
# 使用 kubernetes-dashboard Chart 部署名为 `kubernetes-dashboard` 的 Helm Release
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
```

> 注意执行完成后的输出信息，提示如何开启访问。
``` shell
To access Dashboard run:
  kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443

NOTE: In case port-forward command does not work, make sure that kong service name is correct.
      Check the services in Kubernetes Dashboard namespace using:
        kubectl -n kubernetes-dashboard get svc
```

3. 开启外网访问。  

> 注意网站是https访问。

有2种方式：

方式一：安装完成后提示的方式，但是需要增加`--address`开放访问，否则只有127.0.0.1。  

这个方式为前台启动，简单调试可以nohup转到后台。  

``` shell
kubectl -n kubernetes-dashboard port-forward --address 0.0.0.0 svc/kubernetes-dashboard-kong-proxy 8443:443

# port-forward: 启动端口转发。
# --address 0.0.0.0: 允许从所有网络接口访问该服务。
# svc/kubernetes-dashboard-kong-proxy: 指定要转发的服务（Service），在这里是 kubernetes-dashboard-kong-proxy。
# 8443:443: 将本地机器的 8443 端口映射到服务的 443 端口。
```

方式二：将服务类型更改为 NodePort  

修改文件，执行命令：  

`kubectl -n kubernetes-dashboard edit svc kubernetes-dashboard-kong-proxy`  

将 type 修改为 NodePort，wq退出后自动生效。  

``` shell
spec:
  # type: ClusterIP
  type: NodePort
```

使用命令：  

`kubectl -n kubernetes-dashboard get svc kubernetes-dashboard-kong-proxy`  

查看随机的端口，使用该端口访问。



# 2. 命令

## 2.1 基础命令

### 2.1.1 查看命名空间

命令：  
```shell
kubectl get ns 
```

执行结果：

```shell 
# kubectl get ns 
NAME              STATUS   AGE
default           Active   24m
kube-node-lease   Active   24m
kube-public       Active   24m
kube-system       Active   24m
tigera-operator   Active   28s
``` 

### 2.1.2 查看pods  

命令：

```shell
kubectl get pods -n calico-system
```

执行结果：

``` shell
# kubectl get pods -n calico-system
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
### 2.1.3 get pods 时显示详细信息

命令：  

``` shell
kubectl get pods -n kube-system -o wide
```

执行结果：

``` shell
# kubectl get pods -n kube-system -o wide
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

## 2.2 待补充
无
