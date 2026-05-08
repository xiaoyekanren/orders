# Containerd

## 安装相关
20240924: 发现可以在github直接下载二进制包，https://github.com/containerd/containerd/releases  
### 安装
如下步骤安装即可
``` shell
# 1. 安装依赖
sudo apt-get install -y software-properties-common
# 2. 添加 Docker 的官方 GPG 密钥
wget https://download.docker.com/linux/ubuntu/gpg && sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg gpg
# 3. 添加 Docker 的稳定版源
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# 4. 安装
sudo apt-get update && sudo apt-get install -y containerd
# 5. 启动 + 自启
sudo systemctl start containerd
sudo systemctl enable containerd
```

### 修改数据目录
``` shell
# 1. 停止containerd
systemctl stop containerd
# 2. 修改文件/etc/containerd/config.toml，即新目录
root = "/data/containerd"
# 3. 将原数据迁移，指向新目录
mv /var/lib/containerd /data/containerd
# 4. 重载systemctl，必须
systemctl daemon-reload
# 5. 启动containerd
systemctl start containerd
```

### 配置代理  
1. 启动代理程序  
假设代理程序地址为http://127.0.0.1:7890  

2. 新增配置文件  
``` shell
# 新建文件夹，用于存放代理文件
mkdir -p /etc/systemd/system/containerd.service.d
# 新建并编辑代理文件
vim /etc/systemd/system/containerd.service.d/http-proxy.conf
# 编辑文件
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890"
Environment="HTTPS_PROXY=http://127.0.0.1:7890"
Environment="NO_PROXY=localhost,127.0.0.1,10.10.0.0/16,10.20.0.0/16"
# 重载 systemctl
systemctl daemon-reload
# 重启服务
systemctl restart containerd
```

3. 重启containerd
``` shell
sudo systemctl start containerd
sudo systemctl enable containerd
```
