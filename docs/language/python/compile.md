# 编译和安装
## on ubuntu20

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

## on Centos7
> apt 和 yum 的包名称不一致。  
> Centos7无法使用yum安装openssl3，要手动改源代码的openssl版本到1.1。

``` shell
# 安装依赖
yum install -y epel-release
yum install -y openssl11-devel zlib zlib-devel readline-devel gcc patch libffi-devel python-devel bzip2-devel ncurses-devel sqlite-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel -y
# 参数
# 下载源代码
# 解压缩
# 编译
cd Python-${python_version}
sed -i 's/PKG_CONFIG openssl /PKG_CONFIG openssl11 /g' configure  # 修改openssl版本
``` 

ps: centos7.9 编译python3.12.4会报错，鬼知道为啥。