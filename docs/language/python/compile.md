# 编译和安装
## on ubuntu20

``` shell
# 安装依赖
apt-get install gcc g++ make libssl-dev openssl zlib1g zlib1g-dev libbz2-dev liblzma-dev sqlite3 libsqlite3-dev libsndfile1 build-essential libreadline-dev llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev python3-enchant -y

# 参数
python_version="3.12.4"
prefix_path="/usr/local/python_${python_version}"

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
除安装依赖的命名不同外，基本同ubuntu

``` shell
# 安装依赖， !! 待补全 !!
yum install -y epel
yum install -y openssl11-devel

# 编译python前要修改openssl的版本
cd <python_src>
sed -i 's/PKG_CONFIG openssl /PKG_CONFIG openssl11 /g' configure

# 其他一致
``` 