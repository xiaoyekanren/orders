# 文本处理

## vim
``` shell
# 替换
:%s/vian/sky/g
```

## sed
用于替换文本。
```shell
# 将所有的viva替换成sky
sed -i -e 's/vian/sky/' abc.txt

# 第15行前注释#
sed -i '15s/^/#/' abc.txt

# 第15行末尾添加” 166.111.80.202”
sed -i '15s/$/ 166.111.80.202/' abc.txt

# 在倒数第二行插入数据
sed -i '$i\bbb' abc.txt
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
windows文件的行的结尾是\n\r，linux结尾是\n。windows的文件传到linux时每一行的结尾就会多出来一个字符\r。
``` shell
# 使用命令查看该符号
cat -A abc.txt
# 可以看到这个\r字符被显示为^M，使用如下面命令替换掉该符号
sed -i 's/\r$//' abc.txt
```


## grep
一般用于过滤文本内容、输出结果。
``` shell 
# 查找当前路径下全部文件的文件内容
grep -rn “文件内容” ./
```

扩展命令：
``` shell
# 将grep -rnl查询的包含文件内容的文件名列出，并做替换操作
grep -rnl '8080/ResourceManagement' ./ | xargs sed -i 's/8.5:8080/8.5:18080/g'
```

## wc
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


## xargs 
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
# xargs -I file  即将ls的结果传给这个“file”的变量
# []dest_path]目标路径

```

## find
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

## cat
用于显示文件内容。
``` shell
# 从第3000行开始，显示1000行。即显示3000~3999行
cat filename | tail -n +3000 | head -n 1000

# 显示1000行到3000行，先输出前3000行，在过滤出1000-3000行
cat filename| head -n 3000 | tail -n +1000  

# tail -n 1000：显示最后1000行
# tail -n +1000：从1000行开始显示，显示1000行以后的
# head -n 1000：显示前面1000行
```


## cut
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

## sort
用于将传入内容排序。
``` shell  
# -u 去重复行
# -n 已数值类型去排序
# -r 降序，默认升序
# -t 指定分隔符，一般和 -k 连用
# -k 指定按哪一列排序，可以加多个
```

## tr
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


## 扩展
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

