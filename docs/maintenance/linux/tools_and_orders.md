# 常用工具和命令

## 查看内存
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

## 查看网卡
查看当前网卡是千兆还是百兆
``` shell
ethtool eth1
```

## 查看硬盘
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

## 查看操作系统版本
``` shell
# Centos
cat /etc/redhat-release
# Ubuntu
cat /etc/issue
lsb_release -a
```

## 查看操作系统内核版本
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
 - 使用uuid挂载
``` shell 
# 1. 获取UUID
blkid /dev/sdx1
# 2. 挂载
UUID=11234565	/data	ext4	defaults	0	0
# 硬盘分区UUID 挂载点 文件系统 挂载选项 dump fsck
```
挂载选项：defaults 表示使用系统默认的挂载选项，通常包括 rw（读写）、suid（允许执行文件的用户ID和组ID设置）、dev（解释字符或块特殊设备）、exec（允许执行二进制程序）、auto（可以被 mount -a 自动挂载）、nouser（只有超级用户可以挂载文件系统）和 async（所有的 I/O 到文件系统应该是异步的）。  

dump：用于备份程序。0 = 不备份；1 = 备份。这个功能现在很少使用。  

fsck 顺序：指定启动时文件系统检查的顺序。0 表示不检查，1 表示首先检查（只有根文件系统 / 设置 1），>= 2 表示多块盘时检查的顺序。  


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


