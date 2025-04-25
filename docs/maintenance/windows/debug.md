# 软件安装

## visual c++ 20xx redistributable 安装
更新日期：2024-07-26  
打开网址：https://learn.microsoft.com/zh-cn/cpp/windows/latest-supported-vc-redist  
老软件一般下载2010即可，新软件看报错。

## CMD下设置JDK环境变量
``` shell
setx JAVA_HOME "C:\Progra~1\Java\jdk1.8.0_121"
setx PATH "%PATH%;C:\Progra~1\Java\jdk1.8.0_121\bin;C:\Progra~1\Java\jdk1.8.0_121\jre\bin"
# setx表示配置永久环境变量
# set表示在当前cmd配置环境变量
# Progra~1代表C:\Program Files
# Progra~2代表C:\Program Files (x86)
``` 

## 路由规则设置
``` shell
# 清除已添加的非默认路由
route -f 

# VPN进入局域网，想访问其他网络的话。
route add 0.0.0.0 mask 0.0.0.0 192.168.130.1  # （本地网关）
route add 166.111.0.0 mask 192.168.3.254  # （VPN的网关）
# -p（永久路由）
 
# 删除路由
route del -net 192.168.122.0 netmask 255.255.255.0
删除的时候不用写网关
``` 

## 配置JDK环境变量
``` shell
# JAVA_HOME 
C:\Program Files\Java\jdk1.8.x
# Path 
%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
```


## Windows10-开机自启Chrome，全屏打开指定网站
``` shell
# 因为Win10将开始目录的，启动文件夹弄没了，手动打开目录
explorer C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
# 将开机自启文件仍里面就行
``` 

已chrome为例，假设我们要开机启动chrome，并全屏打开指定网页，
将chrome的快捷方式扔到如上所说文件夹，修改目标为如下，--kiosk是chrome的全屏打开
"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --kiosk http://localhost:3000/
