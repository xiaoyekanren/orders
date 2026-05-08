---
category: 常用软件
createTime: 2025-11-20
---

# clamav
[**clamav**](https://www.clamav.net/) 是一个免费的可以在服务器上安装的杀毒软件，也可以在windows上使用。  

## 1. 安装
ubutnu安装如下：
``` shell
apt install clamav

```


## 2. 配置和说明
clamav 存在2个主要命令和1个服务：
1. freshclam，更新病毒库
2. clamscan， 扫描磁盘
3. clamav-freshclam.service，更新病毒库


### 2.1 freshclam
安装 clamav 会自动创建用户 clamav。

貌似不同的操作系统下，freshclam的配置文件目录不一致，内容也不一致。需要使用 `man freshclam`，翻到最后面查看配置文件具体路径。已知了2种:
> **ubuntu24**: `/etc/clamav/freshclam.conf`
> **kylin v10 sp3**: `/usr/local/etc/freshclam.conf` 


kylin v10下如果以下目录不存在的话，需要创建然后转给 clamav 用户：
配置文件路径：`/usr/local/etc/freshclam.conf` or `/etc/clamav` 
病毒库文件夹：`/var/lib/clamav/` 
日志文件：`/var/log/freshclam.log`  

``` shell
# conf
mkdir -p /usr/local/etc
touch /usr/local/etc/freshclam.conf
chown -R clamav:clamav /usr/local/etc/freshclam.conf

# db path
mkdir -p /var/lib/clamav
chown -R clamav:clamav /var/lib/clamav

# log
mkdir -p /var/log/clamav
touch /var/log/freshclam.log
chown -R clamav:clamav /var/log/clamav
chown -R clamav:clamav /var/log/freshclam.log

```

kylin v10 还需要手动写配置参数，参考如下：
``` shell
# /usr/local/etc/freshclam.conf
DatabaseDirectory /var/lib/clamav
UpdateLogFile /var/log/freshclam.log
LogFileMaxSize 2M
LogVerbose no
LogRotate yes
DatabaseOwner clamav

# 数据库镜像
# DatabaseMirror http://192.168.99.33:8080  # 离线+多台的话，可以配置镜像源，nginx开启目录访问即可
DatabaseMirror db.local.clamav.net
DatabaseMirror database.clamav.net

# 更新设置
Checks 24
MaxAttempts 5
ScriptedUpdates no
``` 


## 3. 命令使用

### 3.1 freshclam
- 用于更新病毒库

``` shell
# 手动更新病毒库
freshclam

# 详细模式更新
freshclam --verbose

# 强制更新，即使版本相同
freshclam --force

# 检查更新但不下载
freshclam --check

# 显示帮助信息
freshclam --help

```

### 3.2 clamscan
- 主要扫描工具
``` shell
# 扫描单个文件
clamscan file.txt

# 递归扫描目录
clamscan -r /home/username/

# 扫描当前目录
clamscan -r .

# 扫描并显示感染文件详情
clamscan -r -i /path/to/scan

# 扫描时发现病毒发出警告声
clamscan -r --bell -i /path/to/scan

# 扫描并删除感染文件（谨慎使用）
clamscan -r --remove /path/to/scan

# 扫描并将感染文件移动到隔离目录
clamscan -r --move=/var/quarantine /path/to/scan

# 扫描并删除感染文件，跳过权限错误
clamscan -r --remove --skip-permission-errors /path/to/scan

# 扫描大文件（最大500MB）
clamscan -r --max-filesize=500M --max-scansize=500M /path/to/scan

# 排除某些目录或文件类型
clamscan -r --exclude-dir="^/proc" --exclude-dir="^/sys" --exclude="\.tmp$" /

# 扫描时记录详细日志
clamscan -r -l /var/log/clamav/daily_scan.log --log-verbose /path/to/scan

# 扫描压缩文件内部
clamscan -r --scan-archive=yes /path/to/scan

# 限制递归深度
clamscan -r --max-dir-recursion=5 /path/to/scan

```

## 4. 其他

### 4.1 扩展命令
#### 4.1.1 clamconf
- 配置检查工具（需要 `apt-get install clamav-daemon` ）

``` shell
# 显示当前配置
clamconf

# 生成默认配置文件
clamconf -g freshclam.conf

# 检查配置文件语法
clamconf -c /etc/clamav/clamd.conf

```

#### 4.1.2 clamdscan 
- 使用守护进程扫描更快（需要 `apt-get install clamdscan` ）

``` shell
# 使用守护进程扫描
clamdscan /path/to/scan

# 递归扫描
clamdscan -r /path/to/scan

# 多线程扫描
clamdscan --multiscan /path/to/scan

# 扫描并删除感染文件
clamdscan --remove /path/to/scan

# 扫描并将感染文件移动到隔离区
clamdscan --move=/var/quarantine /path/to/scan
``` 

### 4.2 定时任务
1点30更新病毒库，2点运行扫描
``` shell
# crontab -e
30 1 * * * freshclam
0 2 * * * /usr/local/bin/clamscan -r -i --move="/var/lib/clamav/quarantine" 
```


## 5. 脚本

``` shell
#!/bin/bash
# 保存为 /usr/local/bin/clamav_scan.sh

LOG_FILE="/var/log/clamav/system_scan_$(date +%Y%m%d).log"
QUARANTINE_DIR="/var/quarantine"
EMAIL="admin@example.com"

# 创建隔离目录
mkdir -p $QUARANTINE_DIR

echo "开始系统扫描: $(date)" | tee -a $LOG_FILE

# 执行扫描
clamscan -r -l $LOG_FILE --move=$QUARANTINE_DIR --bell -i /home /etc /opt /usr /var

# 检查扫描结果
INFECTED_COUNT=$(grep "Infected files" $LOG_FILE | awk '{print $3}')

if [ "$INFECTED_COUNT" -ne "0" ]; then
    echo "发现 $INFECTED_COUNT 个感染文件！" | tee -a $LOG_FILE
    # 发送邮件通知
    echo "发现病毒文件，请检查日志: $LOG_FILE" | mail -s "ClamAV 扫描警报" $EMAIL
else
    echo "扫描完成，未发现感染文件。" | tee -a $LOG_FILE
fi

echo "扫描结束: $(date)" | tee -a $LOG_FILE

```

## 6. 本地病毒库镜像源
参考[官方文档](https://docs.clamav.net/appendix/CvdPrivateMirror.html)安装。

### 4.4.1 安装
依赖：
- python >= 3.6 


安装命令如下：
``` shell
apt-get install python3 python3-pip

pip3 install cvdupdate
cvd config set --dbdir <path-to-dbdir>
cvd update

```
配置http服务，允许 wget 下载即可。

### 4.4.2 配置客户端

``` shell
# Replace `mirror.mylan` and `8000` with your domain and port number.
DatabaseMirror http://127.0.0.1:80

# exec 
freshclam
```

