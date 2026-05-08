# CentOS 7 升级 glibc

> torch > 2.5.1 不再支持旧版 glibc，需升级到 2.28。

## 查看当前版本

``` shell
strings /lib64/libc.so.6 | grep ^GLIBC_
```

## 步骤

### 1. 切换阿里云源

``` shell
cd /etc/yum.repos.d
rm -rf *
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
yum clean all && yum makecache
```

### 2. 安装依赖

``` shell
yum install vim wget texinfo -y
yum groupinstall "Development tools" -y
```

### 3. 安装 devtoolset-8（gcc 8）

CentOS 官方源已失效，使用阿里云源：

``` shell
cat > /etc/yum.repos.d/CentOS-scl.repo << EOF
[centos-sclo-rh]
name=CentOS-7 - SCLo rh
baseurl=https://mirrors.aliyun.com/centos/7/sclo/x86_64/rh/
gpgcheck=0
enabled=1
EOF

yum clean all && yum makecache
yum install scl-utils scl-utils-build -y
yum install devtoolset-8-gcc devtoolset-8-gcc-c++ devtoolset-8-binutils -y

# 验证
scl enable devtoolset-8 bash
gcc --version
```

### 4. 升级 make（需要 4.0 - 4.4）

``` shell
wget http://ftp.gnu.org/pub/gnu/make/make-4.3.tar.gz
tar zxvf make-4.3.tar.gz
cd make-4.3
./configure --prefix=/usr/local/make_4_3
make -j$(nproc) && make install

export PATH=/usr/local/make_4_3/bin:$PATH
```

### 5. 编译 glibc-2.28

``` shell
wget http://ftp.gnu.org/pub/gnu/glibc/glibc-2.28.tar.gz
tar zxvf glibc-2.28.tar.gz
cd glibc-2.28
mkdir build && cd build

scl enable devtoolset-8 bash
MAKE=/usr/local/make_4_3/bin/make ../configure \
  --prefix=/usr/local/glibc_2_28 \
  --with-headers=/usr/include \
  --enable-shared \
  --enable-static \
  --disable-werror \
  --disable-tests \
  --enable-stack-protector=strong

MAKE=/usr/local/make_4_3/bin/make make -j$(nproc) && MAKE=/usr/local/make_4_3/bin/make make install
```

## 常见问题

### 重启后 locale 警告

```
-bash: warning: setlocale: LC_TIME: cannot change locale (en_US.UTF-8)
```

解决：
``` shell
localedef -i en_US -f UTF-8 en_US.UTF-8
```

## 注意事项

- glibc 是核心库，升级可能导致系统不稳定
- 建议使用容器或虚拟机运行需要新版 glibc 的应用
- 可通过 `patchelf` 修改应用的 RPATH 使用独立安装的 glibc