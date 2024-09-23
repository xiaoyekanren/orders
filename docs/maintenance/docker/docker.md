# Docker
## 普通命令
### 构建
``` shell
docker build -t apache/iotdb:0.9.1 .

# eg: 
docker build -t apache/iotdb:0.12.4-node . -f Dockerfile-0.12.4-node
```
### 本地运行
``` shell
docker run -d -p 6667:6667 -p 31999:31999 -p 8181:8181 apache/iotdb:0.12.4-node
# -d 是后台启动
# -t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上
# -i 则让容器的标准输入保持打开。

# 进入容器
docker exec -it <CONTAINER ID> /bin/bash
```
### 查看容器详细信息
``` shell
docker inspect
# 可以查看IP
```

## 发布
### 登录
``` shell
docker login --username=yourhubusername 
```
### 打tag
``` shell
docker tag 6a6365cd99d0  apache/iotdb:0.12.4-node
```
### 上传
``` shell
docker push apache/iotdb:0.12.4-node
```
## 发布跨平台镜像
### 确定引用镜像可以跨平台
例如JAVA，需要去dockerhub寻找支持可以跨平台的jdk
``` shell
FROM --platform=$TARGETPLATFORM eclipse-temurin:11-jre-focal

# eclipse-temurin:11-jre-focal 支持多平台
```
### 安装buildx：
确定可以使用buildx，否则要升级docker
``` shell
docker buildx version
```
指定buildx使用docker-container
``` shell
docker buildx create --name mybuilder --driver docker-container
docker buildx use mybuilder
```
开启用于多平台镜像构建的镜像
``` shell
docker run --rm --privileged tonistiigi/binfmt:latest --install all 
```
### 构建并上传
``` shell
# apache/iotdb:latest
docker buildx build --platform linux/amd64,linux/arm64/v8,linux/arm/v7 -t apache/iotdb:latest -f Dockerfile-0.13.1-node . --push

# apache/iotdb:0.13.1-node
docker buildx build --platform linux/amd64,linux/arm64/v8,linux/arm/v7 -t apache/iotdb:0.13.1-node -f Dockerfile-0.13.1-node . --push
```

## 安装相关
### 安装
官方提供安装脚本，直接使用即可，apt-get install 失败时需要配置 HTTPS_PROXY。
``` shell
wget https://get.docker.com https://get.docker.com -O get-docker.sh 
chmod +x get-docker.sh
./get-docker.sh
``` 
### 修改数据目录
``` shell
# vim /usr/lib/systemd/system/docker.service
# ExecStart行末尾，追加下面内容
 --graph=/data/docker

# 202409，新版docker参数改为了如下：
 --data-root=/data/docker
```

### 配置代理
#### 启动代理程序
假设代理程序地址为http://127.0.0.1:7890
#### 新增配置文件
``` shell
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
```
#### 重启docker
``` shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```



