# github action


## scp 和 ssh
推荐使用 
```
ssh:
https://github.com/marketplace/actions/ssh-remote-commands
scp:
https://github.com/marketplace/actions/scp-files
```
注意事项：  
在github上打开对应仓库->settings->secrets->new repository secret，添加对应的key
```
HOST:主机
HOST_PORT:端口
HOST_USER:用户名
TOKEN:被ssh或scp的服务器私钥
```
同时要在服务器上执行：
```
ssh-keygen -t rsa  # 三次回车
cd ~./ssh
cat id_rsa > authorized_keys2
chmod 640 authorized_keys2
```
就OK了
