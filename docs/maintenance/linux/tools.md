---
category: Linux
---

# 工具使用


## 1. 显卡 GPU
``` shell 
# 查看是否有nvidia的GPU
lspci | grep -i nvidia
```

### 限制功率

1. 临时限制

``` shell
# 给编号0 的显卡限制功率到 400w
sudo nvidia-smi -i 0 -pl 400
# 给编号1 的显卡限制功率到 400w
sudo nvidia-smi -i 1 -pl 400

# -pl 临时限制功率
```

2. 启用持久模式

``` shell
sudo nvidia-smi -i 0 -pm 1
sudo nvidia-smi -i 1 -pm 1

# -pm 1 确保配置持续生效
```

3. 查看功率

``` shell
nvidia-smi -i 0 -q -d POWER
nvidia-smi -i 1 -q -d POWER

# -q -d POWER验证功率设置
```

### 锁定驱动版本

``` shell
apt-mark hold cuda-toolkit-12-9
apt-mark hold nvidia-driver-570-open
```

## 2. 7z
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

## 3. SSH 相关
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

| 工具 | 选项 | 说明 |
|------|------|------|
| sshpass | `-p "密码"` | 明文密码（不推荐） |
| sshpass | `-f 密码文件` | 从文件读取（推荐，权限设 600） |
| rsync | `-a` | 归档模式 |
| rsync | `-v` | 详细输出 |
| rsync | `-z` | 传输压缩 |
| rsync | `-P` | 进度条 + 断点续传 |
| rsync | `--delete` | 删除目标端多余文件 |
| rsync | `-e "ssh -p 端口"` | 指定 SSH 端口 |

#### 使用密码文件同步（推荐）

``` bash
echo "remote_password" > ~/.ssh/remote_pass.txt
chmod 600 ~/.ssh/remote_pass.txt

sshpass -f ~/.ssh/remote_pass.txt rsync -avz /local/dir/ user@remote_ip:/remote/dir/
```

#### 从远程拉取

``` bash
sshpass -f ~/.ssh/remote_pass.txt rsync -avz user@remote_ip:/remote/dir/ /local/dir/
```

#### 非默认 SSH 端口

``` bash
sshpass -f ~/.ssh/remote_pass.txt rsync -avz -e "ssh -p 2222" /local/dir/ user@remote_ip:/remote/dir/
```

#### 首次连接（跳过指纹验证）

``` bash
sshpass -f ~/.ssh/remote_pass.txt rsync -avz -e "ssh -o StrictHostKeyChecking=no -p 22" /local/dir/ user@remote_ip:/remote/dir/
```

#### 批量同步多台服务器

``` bash
for i in {184..206}; do
  sshpass -p "password" rsync -avzu -e "ssh -o StrictHostKeyChecking=no -p 22" /etc/openvpn/client root@11.xxx.17.$i:/etc/openvpn/
done
```

#### 目录尾部斜杠的区别

- 源目录加 `/`：同步目录内内容
- 源目录不加 `/`：同步目录本身

#### 安全提醒

- 生产环境禁止使用 `-p "明文密码"`（会暴露在 history 和 ps 中）
- 优先使用 SSH 密钥认证
- 密码文件权限必须设为 600

## 4. 压力测试
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
+ conv=fsync: 表示把文件的"数据"和"metadata"都写入磁盘（metadata包括size、访问时间st_atime & st_mtime等等），因为文件的数据和metadata通常存在硬盘的不同地方，因此fsync至少需要两次IO写操作。 

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


## 5. SSH 禁用文件传输

限制 SSH 用户只能执行命令，禁止文件传输。

### 方法一：修改用户 Shell

#### 使用 rbash（受限 bash）

```bash
# 将用户的 shell 改为 rbash
usermod -s /bin/rbash username

# 或者创建用户时指定
useradd -s /bin/rbash username
```

#### 自定义受限 Shell

创建一个只允许执行特定命令的脚本：

```bash
# /usr/local/bin/restricted_shell
#!/bin/bash
# 只允许执行的命令列表
allowed_commands="ls cat grep pwd echo"

# 获取用户输入的命令
cmd="$1"

# 检查是否在允许列表中
for allowed in $allowed_commands; do
    if [ "$cmd" = "$allowed" ]; then
        exec "$@"
    fi
done

echo "Command not allowed"
exit 1
```

```bash
chmod +x /usr/local/bin/restricted_shell
usermod -s /usr/local/bin/restricted_shell username
```

### 方法二：修改 sshd_config

#### 禁用 SFTP 子系统

编辑 `/etc/ssh/sshd_config`，注释或删除：

```
#Subsystem sftp /usr/lib/openssh/sftp-server
```

#### 使用 ForceCommand 限制用户

在 `/etc/ssh/sshd_config` 中添加：

```
Match User username
    ForceCommand /bin/bash
    PermitTTY yes
```

#### 禁用端口转发和 X11

```
Match User username
    AllowTcpForwarding no
    X11Forwarding no
    PermitTunnel no
```

### 方法三： authorized_keys 限制

在用户的 `~/.ssh/authorized_keys` 中，在公钥前添加限制：

```
command="/bin/bash",no-port-forwarding,no-X11-forwarding,no-agent-forwarding ssh-rsa AAAA...
```

### 方法四：通过 Group 匹配限制

```bash
# 创建受限用户组
groupadd sftp-disabled

# 将用户加入组
usermod -aG sftp-disabled username
```

在 `/etc/ssh/sshd_config` 中：

```
Match Group sftp-disabled
    ForceCommand /bin/bash
    AllowTcpForwarding no
    X11Forwarding no
```

### 验证

```bash
# 重启 sshd
systemctl restart sshd

# 测试 SCP（应失败）
scp test.txt username@host:/tmp/

# 测试 SFTP（应失败）
sftp username@host

# 测试 SSH（应正常）
ssh username@host
```

### 注意事项

- 方法互斥，选择一种即可
- ForceCommand 会限制所有命令，谨慎使用
- rsync 依赖 SSH，禁用 SCP 后 rsync 也会受影响
