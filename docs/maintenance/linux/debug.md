# 故障排查


## 硬盘故障，/dev/sd* recovering journal
开启无法进系统，报错 ```/dev/sdx recovering journal```，如下图：  
![alt text](20240716001920.webp) 
多半是这块硬盘出问题了，输入root密码登录进去。  
 - 不是系统盘，可以在 ```/etc/fstab``` 下注释掉对应的挂载行。  
 - 是系统盘，考虑使用 ```fstab -y /``` 来尝试修复。  
 - 都不行，尝试关掉自检 ```tune2fs –c 0 –i 0 /dev/sdx``` 在重启。


