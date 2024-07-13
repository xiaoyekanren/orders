# Docker
## 普通命令
### 构建
```
docker build -t apache/iotdb:0.9.1 .

# eg: 
docker build -t apache/iotdb:0.12.4-node . -f Dockerfile-0.12.4-node
```
### 本地运行
```
docker run -d -p 6667:6667 -p 31999:31999 -p 8181:8181 apache/iotdb:0.12.4-node
# -d 是后台启动
# -t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上
# -i 则让容器的标准输入保持打开。

# 进入容器
docker exec -it <CONTAINER ID> /bin/bash
```
### 查看容器详细信息
```
docker inspect
# 可以查看IP
```

## 发布
### 登录
```
docker login --username=yourhubusername 
```
### 打tag
``` 
docker tag 6a6365cd99d0  apache/iotdb:0.12.4-node
```
### 上传
```
docker push apache/iotdb:0.12.4-node
```
## 发布跨平台镜像
### 确定引用镜像可以跨平台
例如JAVA，需要去dockerhub寻找支持可以跨平台的jdk
```
FROM --platform=$TARGETPLATFORM eclipse-temurin:11-jre-focal

# eclipse-temurin:11-jre-focal 支持多平台
```
### 安装buildx：
确定可以使用buildx，否则要升级docker
```
docker buildx version
```
指定buildx使用docker-container
```
docker buildx create --name mybuilder --driver docker-container
docker buildx use mybuilder
```
开启用于多平台镜像构建的镜像
```
docker run --rm --privileged tonistiigi/binfmt:latest --install all 
```
### 构建并上传
```
# apache/iotdb:latest
docker buildx build --platform linux/amd64,linux/arm64/v8,linux/arm/v7 -t apache/iotdb:latest -f Dockerfile-0.13.1-node . --push

# apache/iotdb:0.13.1-node
docker buildx build --platform linux/amd64,linux/arm64/v8,linux/arm/v7 -t apache/iotdb:0.13.1-node -f Dockerfile-0.13.1-node . --push
```

## 配置代理
### 启动代理程序
假设代理程序地址为http://127.0.0.1:7890
### 新增配置文件
```
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
### 重启docker
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```



