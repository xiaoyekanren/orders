---
category: Linux
---

# 命令参考


## 1. shell常用的变量
``` shell
$0  # 当前脚本的名字
$n  # 传递给脚本或者函数的参数，n表示第几个参数
$#  # 传递给脚本或函数的参数个数
$*  # 传递给脚本或函数的所有参数
$@  # 传递给脚本或者函数的所有参数
$$  # 当前shell脚本进程的PID
$?  # 函数返回值，或者上个命令的退出状态
$BASH  # BASH的二进制文件问的路径
$BASH_ENV  # BASH的启动文件
$BASH_VERSINFO[n]  # BASH版本信息，有六个元素
$BASH_VERSION  # BASH版本号
$EDITOR  # 脚本所调用的默认编辑器
$EUID  # 当前有效的用户ID
$FUNCNAME  # 当前函数名
$GROUPS  # 当前用户所属组
$HOME  # 当前用户家目录
$HOSTTYPE  # 主机类型
$LINENO  # 当前行号
$OSTYPE  # 操作系统类型
$PATH  # PATH路径
$PPID  # 当前shell进程的父进程ID
$PWD  # 当前工作目录
$SECONDS  # 当前脚本运行秒数
$TMOUT  # 不为0时，超过指定的秒将退出shell
$UID  # 当前用户ID
```

## 2. 脚本自身的绝对路径
``` shell
SELF=$(cd $(dirname $0); pwd -P)/$(basename $0)
```

## 3. 启用alias
shell里面的alias默认禁用。
``` shell
shopt -s expand_aliases  # 开启
alias abc="lsb_release -a"
abc
```


## 4. 检查变量是否存在
### 变量
如果 varName 不为空，则此语句相当于 $varName;  
如果 varName 未定义，此语句将返回一个错误，并显示"?"定义的错误信息。  

```shell
# 用于函数内部
${varName? Error: The varName is not defined}
# 用于函数外部
${varName:? Error: The varName is not defined}

# 如果 varName 为空，此语句将返回一个错误，并显示"?"定义的错误信息
abc=123
echo ${abc:? Error: no abc.}
unset abc
echo ${abc:? Error: no abc.}
```


### if

``` shell
a=
if [ ! -n "$a" ]; then  
echo "IS NULL"
else
echo "NOT NULL"
fi
#  -n 可加可省略
``` 

###  test

``` shell
a=
if test -z "$a" ; then
echo "a is not set!"
else
echo "a is set !"
fi
```

### 空值判断

``` shell
a=
if [ "$a" = "" ]; then
echo "a is not set!"
else
echo "a is set !"
fi

```

## 5. 常用脚本

### 定时删除文件
<details>
<summary>点击查看代码</summary>

``` shell
#/bin/bash
# 文件夹路径
backup_path='/opt/data/backups'
# 保留的文件数量
no_delete_quantity=30
# 检查间隔
check_interval=43200
current_path=`pwd`
# 判断文件夹内文件的数量，大于保留文件的数量就删，小于就等待下次判断
check(){
	file_count_current=`ls -l $backup_path|grep '^-'|wc -l`
	used_space=`du -Ssh $backup_path|awk '{print $1}'`
	if (($file_count_current <= $no_delete_quantity));then
		echo "`date "+%Y-%m-%d %H:%M:%S"`,Now there are $file_count_current files,$used_space is used,no need to delete"
		continue
	else
		execute
	fi
}
# 删除
execute(){
	file_need_delete=`expr $file_count_current - $no_delete_quantity`
	# 核心，按照时间排序→xargs删除
	ls -l $backup_path|grep "^-"|sort -k 6,7|head -n $file_need_delete|awk '{print $9}'|xargs rm -rf
	echo "`date "+%Y-%m-%d %H:%M:%S"`,Now there are $file_count_current files,Already delete $file_need_delete files"
}
# 输出pid
echo "pid=$$"
# 主循环，睡眠→进目录→判断→是否执行→回之前目录→睡眠
while true;do
	sleep $check_interval
	cd $backup_path	
	check
	cd $current_path
done
```
</details>

## 6. vim
``` shell
# 替换
:%s/vian/sky/g
```

## 7. sed
用于替换文本。
```shell
# 将所有的viva替换成sky
sed -i -e 's/vian/sky/' abc.txt

# 指定行前插入
sed -i '203i# 指定行后插入
sed -i '203a
# 第15行前注释#
sed -i '15s/^/#/' abc.txt
# 第15行末尾添加" 166.111.80.202"
sed -i '15s/$/ 166.111.80.202/' abc.txt

# 查看文件指定行数5-10
sed -n '5,10p' /etc/passwd 
```

扩展使用：
``` shell 
# 将.c和.h文件里的aaa替换为bbb
find ./ -name "*.[ch]" |xargs sed -i 's/aaa/bbb/g' 

# 将第一个字母是d的行替换为11111
sed -i "/^d/c11111" abc.txt
# ^d 指d开头的这一行
# /c 是替换 
```

### 替换windows的换行符
windows文件的行的结尾是
，linux结尾是
。windows的文件传到linux时每一行的结尾就会多出来一个字符。
``` shell
# 使用命令查看该符号
cat -A abc.txt
# 可以看到这个字符被显示为^M，使用如下面命令替换掉该符号
sed -i 's/$//' abc.txt
```


## 8. grep
一般用于过滤文本内容、输出结果。
``` shell
grep "被查找的字符串" 文件名
# –e "正则表达式"
# –i "不区分大小写"
# -c "只显示有多少行，不显示内容"
# –v "输出没有指定字符串的其他行"
```


扩展命令：
``` shell
# 只列出当前目录的文件夹信息
ls -l ./ | grep ^d 

# 只列出文件夹那一列
ls -l ./ | grep ^d | awk '{print $9}'

# 将grep -rnl查询的包含文件内容的文件名列出，并做替换操作
grep -rnl '8080/ResourceManagement' ./ | xargs sed -i 's/8.5:8080/8.5:18080/g'
```

## 9. wc
用于统计文件、内容行数。
``` shell
wc aaa.sh
> 4 7 51 aaa.sh
# 4是行
# 7是单词
# 51是字节

# -c, --bytes 字节
# -m, --chars 字符
# -l, --lines 行
# -w, --words 单词(空白、跳格或换行字符分隔的字符串)
# -L, 打印最长行的长度
```


## 10. xargs 
一般用于管道符|后，用于处理管道符前的内容。
### 删除文件过多
rm -rf删除文件过多时报错```/bin/rm: argument list too long```。
``` shell
# 每次删除9个，直到全部删除
ls | xargs -n 9 rm -rf 

# -n 9 每9个为一行
```
### mv文件数量过多
``` shell
ls [src_path] | xargs -I file  mv file [dest_path]

# [src_path]  一般为./
# xargs -I file  即将ls的结果传给这个"file"的变量
# []dest_path]目标路径

```

## 11. find
用于查找文件。
``` shell
find ./ -name '' -type d  # 只搜索文件夹
find ./ -name '' -type f  # 只搜索文件
find ./ -iname ''  # 忽略大小写
find ./ ! -name "*.txt"  # 后缀名不是txt的文件
find ./ -type f -name "*.txt" -delete  # 删除类型为txt的文件
find ./ -type f -perm 777  # 查找类型是777的文件
find ./ -type f -size +10k  # 查找文件大小>10K的文件
find ./ -type f -size 10k  # 查找文件大小=10K的文件
find . -maxdepth 3 -type f  # 向下最大深度限制为3
find . -mindepth 2 -type f  # 搜索出深度距离当前目录至少2个子目录的所有文件
```

## 12. tail
``` shell
# 查看文件后10行
tail -10 /etc/passwd
# 或
tail -n 10 /etc/passwd 
# 监视某个文件
tail -f /var/log/messages  # 文件被删除后会报错
tail -F /var/log/messages  # 文件被删除后，会等待同名文件出现
```

## 13. head
``` shell
查看文件前5行
head -5 /etc/passwd 
```

## 14. cat
用于显示文件内容。
``` shell
# 从第3000行开始，显示1000行。即显示3000~3999行
cat filename | tail -n +3000 | head -n 1000

# 显示1000行到3000行，先输出前3000行，在过滤出1000-3000行
cat filename| head -n 3000 | tail -n +1000  

# tail -n 1000：显示最后1000行
# tail -n +1000：从1000行开始显示，显示1000行以后的
# head -n 1000：显示前面1000行
```

## 15. cut
用于拆字符串。

``` shell
echo $PATH
> /bin:/usr/bin:/sbin:/usr/sbin:/usr/local/bin:/usr/X11R6/bin:/usr/games
#  1  |    2   |  3  |    4    |       5      |       6      |     7

# 取第五个PATH变量
echo $PATH | cut -d ':' -f 5
> /usr/local/bin

# -d 后面接分隔用字符
# -f 依据 -d 的分隔字符将一段信息分割成为数段，用 -f 取出第x段。
```

## 16. sort
用于将传入内容排序。
``` shell  
# -u 去重复行
# -n 已数值类型去排序
# -r 降序，默认升序
# -t 指定分隔符，一般和 -k 连用
# -k 指定按哪一列排序，可以加多个
```

## 17. tr
用于字符的删除和替换。
``` shell
# -d：删除指令字符
# -c：反选设定字符
# -s: 将连续重复字符删减掉只剩1个


# 将小写字母替换为大写
cat test.txt ｜ tr 'a-z' 'A-Z'
tr 'a-z' 'A-Z' < test.txt

# 删除所有的小写字母
cat test.txt ｜ tr -d 'a-z'
```


## 18. 扩展
### 拼接绝对路径（find du sort tail cut xargs echo pwd）
``` shell
find ./ -name "*.tsfile" | xargs du -s | sort -n -k 1 | tail -n 10 | cut -f 2 | xargs -I file echo `pwd`/file 

# find, 查找后缀为tsifle的文件
# du, 输出每个文件大小
# sort, 按大小排序
# tail, 取前10个文件，即最大的10个文件
# cut, 用于裁剪du的输出结果，取文件相对路径
# xargs -I file, 把每行的内容赋给参数file
# echo, 打印绝对路径
```

### 拿到硬盘的UUID（blkid awk awk sed）
blkid拿到的行内容过多，在写脚本的时候只需要uuid，可通过如下方式过滤取到UUID。
``` shell
blkid /dev/vdc1 
> /dev/vdc1: UUID="35abbb5d-a8a9-47d8-8ffc-c4b5d85d4813" TYPE="ext4" PARTLABEL="primary" PARTUUID="e9ac9a6f-e812-4df6-98e7-eb3ac67c67e2"

blkid /dev/vdc1  | awk '{print $2}'| awk -F = '{print $2}' |sed 's:"::g'
# 取UUID         # 取第二列，空格分割 # 取第二列，=分割      #删除双引号
> 35abbb5d-a8a9-47d8-8ffc-c4b5d85d4813
```


## 19. 查看 系统/硬件 信息

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

## 20. 进程绑定CPU
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


## 21. 压缩
### tar

### gzip

### zip/unzip

## 22. lsof
### 查看已经删除，尚未释放的文件
可以看到被打上deleted的文件，在pid被关闭的时候就会释放。用于debug删除文件但是磁盘空间未释放。
``` shell
lsof |grep deleted
```


## 23. 端口监听 nc
nc：Netcat 工具，用于读写网络连接。

``` shell
nc -lk 0.0.0.0 6066

# -l：启用监听模式，等待传入连接。
# -k：允许 Netcat 在连接关闭后继续监听（保持监听状态）。
# 0.0.0.0：监听所有可用的网络接口。
# 6066：指定监听的端口号。
```
如上，即可监听本地的6666端口，这时候可以使用telnet，随便输入数据，那边实时接收。

## 24. 查看程序启动路径
pwdx

## 25. 环境变量加载顺序

1. 全局配置   
当用户登录时，系统首先执行 `/etc/profile`。这个文件通常用于设置全局环境变量和启动程序。  
然后，`/etc/profile` 可能会调用 `/etc/bashrc`（或其他配置文件），用于设置交互式 shell 的环境。  

2. 用户配置
接下来，用户的个人配置文件 `~/.bash_profile` 被执行。这个文件通常用于设置用户的特定环境变量和启动程序。
`~/.bash_profile` 可能会调用 `~/.bashrc`，以确保交互式 shell 的环境变量和设置都被加载。  

3. 环境变量覆盖  
如果在 `/etc/profile` 和 `~/.bash_profile` 中定义了同名的环境变量，后者（即 `~/.bash_profile` 中的定义）会覆盖前者。这是因为后加载的变量会替代先加载的变量的值。
