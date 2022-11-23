## 构建
```
docker build -t apache/iotdb:0.9.1 .
# eg: 
docker build -t apache/iotdb:0.12.4-node . -f Dockerfile-0.12.4-node

```
## 本地运行
```
docker run -d -p 6667:6667 -p 31999:31999 -p 8181:8181 apache/iotdb:0.12.4-node
docker exec -it <CONTAINER ID> /bin/bash

```


# 发布
## 登录
```
docker login --username=yourhubusername 
```
## 
``` 
docker tag 6a6365cd99d0  apache/iotdb:0.12.4-node
```

## 上传
```
docker push apache/iotdb:0.12.4-node
```
