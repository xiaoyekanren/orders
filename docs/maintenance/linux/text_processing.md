# 文本处理

## 替换
将所有的viva替换成sky

```shell
# vim
:%s/vian/sky/g
# sed
sed -i -e 's/vian/sky/' abc.txt
```
### 替换windows的换行符
windows下，每一行的结尾是\n\r，而在linux下文件的结尾是\n。windows的文件传到linux时每一行的结尾就会多出来一个字符\r。
``` shell
# 使用命令查看该符号
cat -A abc.txt
# 可以看到这个\r字符被显示为^M，使用如下面命令删除该符号
# 替换掉该符号
sed -i 's/\r$//' abc.txt
```
