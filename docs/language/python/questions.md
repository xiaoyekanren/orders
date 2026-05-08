---
category: Python
---

# 运维相关

## 编译和安装

### 1. on ubuntu20

``` shell
# 安装依赖
apt-get install gcc g++ make libssl-dev openssl zlib1g zlib1g-dev libbz2-dev liblzma-dev sqlite3 libsqlite3-dev libsndfile1 build-essential libreadline-dev llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev python3-enchant -y

# 参数
python_version="3.12.4"
prefix_path="/usr/local/python_${python_version}"

mkdir -p ${prefix_path}

# 下载源代码
wget https://www.python.org/ftp/python/${python_version}/Python-${python_version}.tgz

# 解压缩
tar zxvf Python-${python_version}.tgz
rm Python-${python_version}.tgz

# 编译
cd Python-${python_version}
./configure --enable-optimizations --prefix=${prefix_path}
make -j8 && make install
```

### 2. on Centos7

> apt 和 yum 的包名称不一致。

> Centos7无法使用yum安装openssl3，要手动改源代码的openssl版本到1.1 or 手动编译。

> 如果手动编译：  
`./configure`要增加`--with-openssl=/usr/local/openssl`，同时`vim /etc/ld.so.conf.d/openssl.conf`,写入`/usr/local/openssl`
> ，执行`ldconfig`

``` shell
# 安装依赖
yum install -y epel-release
yum install -y openssl11-devel zlib zlib-devel readline-devel gcc patch libffi-devel python-devel bzip2-devel ncurses-devel sqlite-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel -y
# 编译
cd Python-${python_version}
sed -i 's/PKG_CONFIG openssl /PKG_CONFIG openssl11 /g' configure  # 修改openssl版本


# 其他一致
``` 

ps: centos7.9 编译python3.12.4会报错，鬼知道为啥。

## 疑难杂症

### 1. Windows 使用 venv 环境失败的问题

问题：PowerShell 无法执行激活脚本

错误信息：

```
.\venv\Scripts\activate : 无法加载文件，因为在此系统上禁止运行脚本
```

1. 修改执行策略（永久生效，推荐）

以管理员身份运行 PowerShell，执行：

```powershell
# 针对当前用户，永久生效
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Process，当前进程，无需管理员
# CurrentUser，当前用户，无需管理员权限
# LocalMachine，所有用户，需管理员权限
```

2. （方案2）绕过执行策略激活

无需修改策略，直接执行：

```powershell
powershell -ExecutionPolicy Bypass -File .\venv\Scripts\activate.ps1
```

3. 激活 windows 虚拟环境

```powershell
# 创建
python -m venv venv

# 激活
.\venv\Scripts\activate

# 退出
deactivate
```
