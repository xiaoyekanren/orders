# shell

## shell常用的变量
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

## 脚本自身的绝对路径
``` shell
SELF=$(cd $(dirname $0); pwd -P)/$(basename $0)
```

## 启用alias
shell里面的alias默认禁用。
``` shell
shopt -s expand_aliases  # 开启
alias abc="lsb_release -a"
abc
```


## 检查变量是否存在
### ${abc :?}
如果 varName 不为空，则此语句相当于 $varName;  
如果 varName 未定义，此语句将返回一个错误，并显示“?”定义的错误信息。  

```shell
# 用于函数内部
${varName? Error: The varName is not defined}
# 用于函数外部
${varName:? Error: The varName is not defined}

# 如果 varName 为空，此语句将返回一个错误，并显示“?”定义的错误信息
abc=123
echo ${abc:? Error: no abc.}
unset abc
echo ${abc:? Error: no abc.}
```


### if

``` shell
a=
if [ ! -n "$a" ]; then  
echo "IS NULL"
else
echo "NOT NULL"
fi
#  -n 可加可省略
``` 

###  test

``` shell
a=
if test -z "$a" ; then
echo "a is not set!"
else
echo "a is set !"
fi
```

### 空值判断

``` shell
a=
if [ "$a" = "" ]; then
echo "a is not set!"
else
echo "a is set !"
fi

```