# 常用工具和命令

## 查看 系统/硬件 信息

### 查看内存
``` shell
# 查看内存详细信息
dmidecode -t memory
# 显示插槽的使用情况
dmidecode|grep -P -A5 "Memory\s+Device"|grep Size|grep -v Range
# 显示最大支持内存数量
dmidecode | grep -P 'Maximum\s+Capacity'
# 查看内存频率
dmidecode|grep -A16 "Memory Device"
dmidecode|grep -A16 "Memory Device"|grep 'Speed'
 ```

### 查看网卡
查看当前网卡是千兆还是百兆
``` shell
ethtool eth1
```

### 查看硬盘
``` shell
# 机械硬盘
apt-get install smartmontools -y
smartctl --all /dev/sda

# nvme
apt-get install nvme-cli
nvme list

# 查看分区
lsblk
```

### 查看操作系统版本
``` shell
# Centos
cat /etc/redhat-release
# Ubuntu
cat /etc/issue
lsb_release -a
```

### 查看操作系统内核版本
``` shell
uname -a
```

##  swap相关

### 删除swap

``` shell
# 1. 查看swap
swapon --show
# 2. 关闭swap
swapoff /dev/dm-1
# 3. 删除当前swap
rm -rf /dev/dm-1
# 4. 修改fstab文件，注释掉swap相关行
vim /etc/fstab
#LABEL=YUNIFYSWAP swap                    swap    defaults        0 0
```

### 创建swap

``` shell
# 1. 使用 fallocate 生成一个文件
fallocate -l 32G /swapfile
# 2. 设置权限为 600
chmod 600 /swapfile
# 3. 将文件标记为 swap
mkswap /swapfile
# 4. 开启 swap
swapon /swapfile
# 5. 开机自启动
vim /etc/fstab
/swapfile        none   swap    sw              0       0
# 6. 查看swap
swapon --show
```


## 清理 cache
在 Linux 系统中，内存管理是通过内核自动完成的。内核会将未使用的内存用作缓存（cache）和缓冲区（buffers）以提高系统性能。缓存和缓冲区是用来临时存储数据，以便快速访问。

缓存（Cache）：通常指的是用来存储读取的文件系统数据的内存区域。当文件被读取时，它们的数据会被存储在缓存中，以便下次访问时能更快地读取。
缓冲区（Buffers）：用来存储即将写入磁盘的数据。缓冲区允许操作系统将多个小的写操作合并成较少的大块操作，这样可以减少对磁盘的操作次数，提高效率。

缓存可按照如下命令清理，缓冲区只可以重启清理。
``` shell
sync  # 第一步

# 第二步，三选一
echo 1 > /proc/sys/vm/drop_caches   # 清理页缓存（page cache）
echo 2 > /proc/sys/vm/drop_caches  # 清理目录项和inode缓存
echo 3 > /proc/sys/vm/drop_caches  # 清理页缓存，目录项和inode缓存
```

## 挂载
### 挂载新硬盘，分区 & 格式化
1. 使用parted分区
``` shell
parted /dev/sdx
mklabel gpt
# mkpart 二选一
mkpart primary 1 -1  # 使用所用空间
mkpart primary 2048s 100%  # 4k对齐
print  # 打印分区信息
quit  # 退出parted
```
2. 格式化
``` shell
mkfs.ext4 /dev/sdx1
```
3. mount 
``` shell
mkdir /data  # 必须
mount /dev/sdx1 /data
```
4. 开机自挂载  
 - 直接使用 ```/dev/sdx1``` 挂载（不推荐，硬盘换位置或者掉了系统就启动不了了）。
``` shell 
# 编辑文件 /etc/fstab，追加行
# vim /etc/fstab
/dev/sdb1	/data	ext4	defaults	0	0
# 硬盘分区 挂载点 文件系统 挂载选项 dump fsck
``` 
|硬盘分区|挂载点|文件系统|挂载选项|备份|启动是否检查|
|:-|:-|:-|:-|:-|:-|
|/dev/sdb1|/data|ext4|defaults|0|0|  

 + 挂载选项
    + defaults 表示使用系统默认的挂载选项，通常包括 rw（读写）、suid（允许执行文件的用户ID和组ID设置）、dev（解释字符或块特殊设备）、exec（允许执行二进制程序）、auto（可以被 mount -a 自动挂载）、nouser（只有超级用户可以挂载文件系统）和 async（所有的 I/O 到文件系统应该是异步的）。  
    + nodiratime：不更新目录的访问时间。
    + noatime：不更新文件的访问时间。
    + 可以写成 defaults,noatime,nodiratime，意思是 使用默认选项，并禁用文件和目录的访问时间更新。
 + dump：用于备份程序。0 = 不备份；1 = 备份。这个功能现在很少使用。  
 + fsck 顺序：指定启动时文件系统检查的顺序。0 表示不检查，1 表示首先检查（只有根文件系统 / 设置 1），>= 2 表示多块盘时检查的顺序。  

5. 使用uuid挂载
``` shell 
# 1. 获取UUID
blkid /dev/sdx1
# 2. 挂载
UUID=11234565	/data	ext4	defaults	0	0
# 硬盘分区UUID 挂载点 文件系统 挂载选项 dump fsck
```


### 外置硬盘
``` shell
# 挂载光盘
ls -l /dev | grep cdrom  # 查看当前光盘
mount /dev/cdrom1 /mnt/gp  #  将光盘cdrom1挂载到/mnt/gp下

# 挂载 windows 共享硬盘
mount -t cifs -o username=administrator,password='123' //192.168.1.181/share /share
```

## 虚拟机不重启添加磁盘
``` shell 
for i in `ls /sys/class/scsi_host/*/scan`; do echo "- - -" > $i; done
```

## 显卡 GPU
``` shell 
# 查看是否有nvidia的GPU
lspci | grep -i nvidia
```


## sudo 权限
给普通用户 sudo 权限，使其当前命令可以以 root 权限来运行。
编辑文件 ```/etc/sudoers```。
``` shell
# 正常情况
ubuntu	ALL=(ALL:ALL) ALL
# sudo 免root密码
ubuntu ALL=(ALL) NOPASSWD:ALL
```

## 网卡配置
### ubuntu16
``` shell
# vim /etc/network/interfices
auto ens33  # 网卡名称，ip add / ifconfig 获得
iface ens33 inet static  # 静态，如果dhcp只配置auto 和iface 2行即可
address 192.168.1.222  # ip
netmask 255.255.255.0  # 子网
gateway 192.168.1.1  # 网关
dns-nameservers 192.168.1.1  # dns，空格分隔配置多个

# 重启网卡

```

### centos7
``` shell
# vim /etc/sysconfig/network-scripts/ifcfg-<enoxxxxx>
DEVICE="eth0"  # 设备名称
BOOTPROTO="static"  # 静态ip
ONBOOT="yes"  # 开机自启
TYPE="Ethernet"  # 网络类型
IPADDR="192.168.1.122"  # IP地址
GATEWAY="192.168.1.1"  # 网关
NETMASK="255.255.255.0"  # 子网掩码
DNS1=192.168.1.1  # dns

# 重启所有
/etc/init.d/netowrk restart

# 重启单个
ifup <网卡>
ifdown <网卡>
```

### ubuntu>20
这个是yaml，一定要注意格式。
``` shell
# vim /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    eno1:  # 网卡
      addresses:
      - 192.168.1.5/24  # IP地址
      gateway4: 192.168.1.254  # 网关
      nameservers:
        addresses:
        - 166.111.8.28  # dns
  version: 2
# 使生效
netplan apply
```

 - 下面是如何配置dhcp
``` shell
network:
  ethernets:
    enp2s0:
      dhcp4: true
  version: 2
```


## 临时修改dns
重启网卡失效！重启失效！重启失效！
``` shell
# vim /etc/resolv.conf
nameserver 114.114.114.114
```

## 防火墙
### centos
``` shell
# 启停
systemctl start firewalld  # 启动
systemctl status firewalld  # 查看状态
systemctl disable firewalld  # 停止
systemctl stop firewalld  # 禁用
firewall-cmd --reload  # 重载防火墙规则

# 配置
firewall-cmd --zone=public --list-ports   查看所有打开的端口
firewall-cmd --get-active-zones           查看区域信息
firewall-cmd --get-zone-of-interface=eth0 查看指定接口所属区域

# 开放/关闭
firewall-cmd --zone=public --list-ports  # 查看所有打开的端口
firewall-cmd --zone=public --query-port=80/tcp  # 查看端口状态

firewall-cmd --zone=public --add-port=8080/tcp --permanent  # 开放端口
firewall-cmd --zone=public --remove-port=80/tcp --permanent  # 关闭端口
  # --permanent 永久生效，没有此参数重启后失效

```

### ubuntu
``` shell
ufw allow 22
ufw allow 22 comment 'Allow SSH connections'  # 带备注
ufw allow proto tcp to 0.0.0.0/0 port 443 comment "this is comment"  # 完整

# 重新加载
ufw enable  # 启用ufw
ufw disable  # 关闭ufw
ufw reload  # 重载参数

# 删除规则
ufw status numbered  # 查看规则的index编号
ufw delete [1]  # 按照编号删除规则
```

## ulimit 相关
ulimit 用于控制用户级别的资源限制

``` shell
# 显示所有的可配置的值
ulimit -a
# 查看单项，例如最大打开文件数量
ulimit -n
# 当前shell生效，调整最大文件打开数量
ulimit -n 102400
```

永久生效需要编辑文件```/etc/security/limits.conf```后重启shell生效。
``` shell
# 设置最大进程数
* soft nproc 102400  # 任何用户可以打开的最大进程数
* hard nproc 102400

# 设置最大文件打开数
* soft nofile 102400  # 任何用户可以打开的最大的文件描述符数量，默认1024，会限制tcp连接
* hard nofile 102400

# soft是一个警告值
# hard是一个真正意义的阀值，超过就会报错
```

注1：* 表示所有用户，@ 代表所有组。  
注2：ubuntu20上，* 不包含root，root需指定root才生效。  
注3：通过 SSH 登录的用户，确保 sshd 守护进程配置中的 UsePAM 设置为 yes，否则不生效。

## 进程绑定CPU
taskset 用于设置或查找一个进程的 CPU 亲和性。  
``` shell
# 指定pid，查看该pid所绑定CPU
taskset -p 2726

# 启动程序时绑定CPU
taskset -c 1 sleep 3  # 在CPU1上运行sleep 3秒

# 改变现有程序绑定的CPU，两种写法都行
taskset -pc 0,3,7-11 2726
taskset -c 0,3,7-11 -p 2726

```
补充：查看绑定CPU的输出结果是十六进制，要将这个值转成二进制，之后从右往左每一位数代表一个CPU，值1表示该pid允许在该CPU运行，值0表示不允许。  
例如1：返回5，二进制是101，CPU0、CPU2允许运行，CPU1不允许运行。  
例如2：返回4，二进制是100，只允许在CPU2上运行。  
例如3：返回f，二进制是1111，允许在CPU0-3上执行。  


## 压缩
### tar

### gzip

### zip/unzip

### 7z
需要安装，安装方式```sudo apt-get install p7zip-full -y```


``` shell
# 压缩
7z a archive.7z file1 file2 dir1/
# a：压缩
# archive.7z：压缩后的文件名
# "file1 file2 dir1/"：压缩的内容

#  解压缩
7z x archive.7z
# x：解压缩

# 其他命令：
# l：表示列出压缩包内容，L的小写
# t：验证文件完整性

# 其他参数：
# -o./：指定路径(后无空格)
# -t7z，指定压缩格式7z(后无空格)
```

## lsof
### 查看已经删除，尚未释放的文件
可以看到被打上deleted的文件，在pid被关闭的时候就会释放。用于debug删除文件但是磁盘空间未释放。
``` shell
lsof |grep deleted
```


## 包管理工具
### ubuntu - apt

#### 查看使用apt安装的文件的位置
``` shell
dpkg -L <package>

# eg:
dpkg -L tree
# /.
# /usr
# /usr/bin
# /usr/bin/tree
# /usr/share
# /usr/share/doc
# /usr/share/doc/tree
# /usr/share/doc/tree/README.gz
# /usr/share/doc/tree/TODO
# /usr/share/doc/tree/changelog.Debian.gz
# /usr/share/doc/tree/copyright
# /usr/share/man
# /usr/share/man/man1
# /usr/share/man/man1/tree.1.gz
```

### centos - yum
cengos7 和 8 都无啦，https://blog.centos.org/2023/04/end-dates-are-coming-for-centos-stream-8-and-centos-linux-7/
#### 切换国内源
阿里云源参考地址：https://developer.aliyun.com/mirror/centos/


## ssh相关
### sshpass
类似于ssh，只是可以指定密码
```shell
# 基础使用
sshpass -p <passowrd> ssh <user>@<ip>
# 远程执行命令
sshpass -p <password> ssh <user>@<ip> "whoami"
```

### rsync
就是scp plus版本
```shell
rsync -avzu -e'ssh -p 55555' /var/lib/mysql ubuntu@192.168.10.10:/mysqlbak
# 等同于
scp -P 55555 -r /var/lib/mysql ubuntu@101.6.15.214:/mysqlbak
```
+ 如果文件过大，需要nohup后台运行的时候，最好是免密验证。
+ 追求速度的话，取消参数z
+ 如果两端有端口有端口映射的情况，建议使用ssh传输，即 -e ssh

``` shell
参数
-a, --archive archive mode 权限保存模式,相当于 -rlptgoD 参数，存档，递归，保持属性等
-z, --compress 压缩模式, 当资料在传送到目的端进行档案压缩
-v , --verbose 复杂的输出信息
-u 跳过目标路径较新的文件
-P 显示传输速度
--delete， 删除那些目标位置有的文件而备份源没有的文件
--password-file=FILE ，从 FILE 中得到密码
```

## 压力测试
### CPU 计算π
通过计算π来进行的压力测试。
``` shell
# 计算50000位圆周率
echo "scale=50000; 4*a(1)" | bc -l -q

# scale=50000：设置小数位数。
# 4*a(1)：使用 bc 的内置函数 a(1) 计算 arctan(1)，即 π / 4 * 4 = π
# |：管道符，将前面 echo 生成的字符串传递给 bc。
# bc：一个任意精度的计算器语言。
# -l：启用数学库，提供数学函数如 a()。
# -q：安静模式，避免显示启动信息。
```

### CPU stress
``` shell
stress --cpu 8
```

### 内存 memtester
使用 memtester 做内存压力测试
``` shell
# 安装
apt-get install memtester -y

# 测试，分配10G内存，测试20次
memtester 10G 20
``` 

### CPU温度监控
``` shell
# CPU温度监控
watch -n1 sensors

# CPU主频监控
watch -n1 "cat /proc/cpuinfo | grep \"^[c]pu MHz\""
```

### 硬盘 dd
使用dd进行磁盘的测试
``` shell
dd if=/dev/zero of=test1 bs=25MB count=80 oflag=direct  # 写
dd if=test1 of=/dev/null iflag=direct  # 读
dd if=/dev/sda of=/testrw.db bs=4k  # 同时读写

# if：指定读取的文件
# of：指定写入的文件
# bs：写入&输出的块大小，ibs=读，obs=写
# count：写入的块数量
# conv=fsync：dd命令执行到最后会执行一次sync
# oflag＝direct：测速貌似要加这个
# /dev/zero：零设备，可提供无限的空字符
```
+ conv=fsync: 表示把文件的“数据”和“metadata”都写入磁盘（metadata包括size、访问时间st_atime & st_mtime等等），因为文件的数据和metadata通常存在硬盘的不同地方，因此fsync至少需要两次IO写操作。 

+ oflag=dsync 每个block size都单独写一次磁盘，使用同步I/O，去除caching的影响，这是最慢的一种方式，可以当成是模拟数据库插入操作。  

+ oflag=direct,nonblock 避掉文件系统cache,直接读写,不使用buffer cache

### 网络 iperf 
``` shell
# server
iperf -s

# client
iperf -c 192.168.130.13 -i 1 -n 10240000000000

# -c：指定服务器ip
# -i：指定报告间隔，即每N秒报告一次
# -n：指定发送包大小，kb
```


## 端口监听 nc
nc：Netcat 工具，用于读写网络连接。

``` shell
nc -lk 0.0.0.0 6066

# -l：启用监听模式，等待传入连接。
# -k：允许 Netcat 在连接关闭后继续监听（保持监听状态）。
# 0.0.0.0：监听所有可用的网络接口。
# 6066：指定监听的端口号。
```
如上，即可监听本地的6666端口，这时候可以使用telnet，随便输入数据，那边实时接收。

## 查看程序启动路径
pwdx


## gcc

### centos7 升级 gcc
升级需谨慎。  
必须按照顺序一个一个编译。  
1. 安装gmp  
下载地址：https://gmplib.org/download/gmp/  

``` shell
./configure --prefix=/usr/local/gmp
make && make install
```

2. 安装MPFR  
下载地址：https://www.mpfr.org/mpfr-current  

``` shell
./configure --prefix=/usr/local/mpfr --with-gmp=/usr/local/gmp
make && make install
```

3. 安装mpc  
下载地址：ftp://ftp.gnu.org/gnu/mpc/  

``` shell
./configure --prefix=/usr/local/mpc --with-gmp=/usr/local/gmp --with-mpfr=/usr/local/mpfr
make && make install
```

4. 安装gcc  
下载地址：ftp://ftp.gnu.org/gnu/gcc/  

``` shell
export LD_LIBRARY_PATH=/usr/local/mpc/lib:/usr/local/gmp/lib:/usr/local/mpfr/lib：$LD_LIBRARY_PATH
./configure --prefix=/usr/local/gcc --enable-threads=posix --disable-checking --disable-multilib --enable-languages=c,c++ --with-gmp=/usr/local/gmp --with-mpfr=/usr/local/mpfr --with-mpc=/usr/local/mpc
make -j$(nproc)&& make install
```

5. 制作软链接  

``` shell
ln -sf /usr/local/gcc/bin/gcc /usr/bin/gcc
ln -sf /usr/local/gcc/bin/c++ /usr/bin/c++
ln -sf /usr/local/gcc/bin/g++ /usr/bin/g++
ln -sf /usr/local/gcc/lib64/libstdc++.so.6.0.22 /usr/lib64/libstdc++.so.6
```

## ubuntu-清理多余的内核
``` shell
# 查看当前内核
uname -a

# 查看所有内核
dpkg --get-selections | grep linux

# 移除多余内核，移除之后给所有内核打上已删除标记
apt-get remove a b c d

# 删除"已删除标记"
dpkg --purge `dpkg --get-selections | grep deinstall | cut -f1`
```

## ubuntu-限制用户登陆
``` shell
# 新增参数
/etc/pam.d/sshd
文件里有这一行
session required pam_limits.so

# 增加限制
/etc/security/limits.conf文件末尾增加：
# 所有用户单用户登录：
* - maxlogins 1
# 指定用户单用户登录：
@user - maxlogins 1

# 重启服务器
```

## linux 环境变量加载顺序

1. 全局配置   
当用户登录时，系统首先执行 `/etc/profile`。这个文件通常用于设置全局环境变量和启动程序。  
然后，`/etc/profile` 可能会调用 `/etc/bashrc`（或其他配置文件），用于设置交互式 shell 的环境。  

2. 用户配置
接下来，用户的个人配置文件 `~/.bash_profile` 被执行。这个文件通常用于设置用户的特定环境变量和启动程序。
`~/.bash_profile` 可能会调用 `~/.bashrc`，以确保交互式 shell 的环境变量和设置都被加载。  

3. 环境变量覆盖  
如果在 `/etc/profile` 和 `~/.bash_profile` 中定义了同名的环境变量，后者（即 `~/.bash_profile` 中的定义）会覆盖前者。这是因为后加载的变量会替代先加载的变量的值。  


## apt代理
`vim /etc/apt/apt.conf.d/proxy.conf`  

``` shell
Acquire {
  http::Proxy "http://127.0.0.1:7890/";
  https::Proxy "http://127.0.0.1:7890/";
}
```

