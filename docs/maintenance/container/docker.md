# Docker

## 1. 普通命令

### 1.1 构建

```shell
docker build -t apache/iotdb:0.9.1 .

# eg:
docker build -t apache/iotdb:0.12.4-node . -f Dockerfile-0.12.4-node
```

### 1.2 运行一个容器

```shell
docker run -d -p 6667:6667 -p 31999:31999 -p 8181:8181 apache/iotdb:0.12.4-node
# -d 是后台启动
# -t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上
# -i 则让容器的标准输入保持打开。

# 进入容器
docker exec -it <IMAGE_ID> /bin/bash

# 直接进入一个容器，例如某个操作系统
docker run -it --entrypoint "/bin/bash" <IMAGE_ID>
```

### 1.3 查看容器详细信息

```shell
docker inspect
# 可以查看IP
```

## 2. 发布到 docker-hub

### 2.1 登录

```shell
docker login --username=yourhubusername
```

### 2.2 打 tag

```shell
docker tag 6a6365cd99d0  apache/iotdb:0.12.4-node
```

### 2.3 上传

```shell
docker push apache/iotdb:0.12.4-node
```

### 2.4 发布跨平台镜像

#### 2.4.1 确定引用镜像可以跨平台

例如 JAVA，需要去 dockerhub 寻找支持可以跨平台的 jdk

```shell
FROM --platform=$TARGETPLATFORM eclipse-temurin:11-jre-focal

# eclipse-temurin:11-jre-focal 支持多平台
```

#### 2.4.2 安装 buildx：

确定可以使用 buildx，否则要升级 docker

```shell
docker buildx version
```

指定 buildx 使用 docker-container

```shell
docker buildx create --name mybuilder --driver docker-container
docker buildx use mybuilder
```

开启用于多平台镜像构建的镜像

```shell
docker run --rm --privileged tonistiigi/binfmt:latest --install all
```

### 构建并上传

```shell
# apache/iotdb:latest
docker buildx build --platform linux/amd64,linux/arm64/v8,linux/arm/v7 -t apache/iotdb:latest -f Dockerfile-0.13.1-node . --push

# apache/iotdb:0.13.1-node
docker buildx build --platform linux/amd64,linux/arm64/v8,linux/arm/v7 -t apache/iotdb:0.13.1-node -f Dockerfile-0.13.1-node . --push
```

## 3. 安装相关

### 3.1 安装

官方提供安装脚本，直接使用即可，apt-get install 失败时需要配置 HTTPS_PROXY。

```shell
wget https://get.docker.com https://get.docker.com -O get-docker.sh
chmod +x get-docker.sh
./get-docker.sh
```

### 3.2 修改数据目录

```shell
# vim /usr/lib/systemd/system/docker.service
# ExecStart行末尾，追加下面内容
 --graph=/data/docker

# 202409，新版docker参数改为了如下：
 --data-root=/data/docker
```

### 3.3 配置代理

1. 启动代理程序
   假设代理程序地址为http://127.0.0.1:7890
2. 新增配置文件

```shell
# 新增/修改 配置文件，写死docker的镜像为官方镜像
# vim /etc/docker/daemon.json
{
 "registry-mirrors": [
    "https://hub.docker.com/"]
}

# 创建文件夹
mkdir /etc/systemd/system/docker.service.d

# 新增文件
# vim /etc/systemd/system/docker.service.d/proxy.conf
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890"
Environment="HTTPS_PROXY=http://127.0.0.1:7890"
Environment="NO_PROXY=localhost,127.0.0.1,10.10.0.0/16,10.20.0.0/16"
```

3. 重启 docker

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 3.4 完整删除 docker

```shell
systemctl stop docker
sudo apt-get purge docker-ce docker-ce-cli containerd.io
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
sudo rm -rf /etc/docker/daemon.json
sudo rm -rf /etc/systemd/system/docker.service
sudo rm -rf /etc/systemd/system/docker.socket

```

## 4. 扩展命令

### 4.1 配置容器开启自启

--restart 有4种策略：
  > no，不自动重启，默认
  > on-failure[:max-retries]，异常退出自动重启
  > always，无论如何都自动重启
  > unless-stopped， 除非手动停止容器或者重启服务，否则总是自动重启


1. 在运行时指定

``` shell
docker run \
  --restart always \
  -d \
  --name nginx \
  nginx
```

2. 当前运行的容器修改
``` shell
docker update --restart unless-stopped auto-start-nginx
```

3. 查看重启策略
``` shell
docker inspect -f "{{ .HostConfig.RestartPolicy.Name }}" 容器名/容器ID
```


