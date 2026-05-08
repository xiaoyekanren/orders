---
category: 常用软件
---

# OpenVPN

## 1. 名词解释

### 1. 推送域名解析

服务端通过 `push "dhcp-option xxx"` 向客户端推送配置。
注意，启用 `push "redirect-gateway def1 bypass-dhcp"` 时，所有流量强制走VPN，此时 `push "dhcp-option DNS xxx"` 会全局生效。关闭时，只对
`push "dhcp-option DOMAIN xxx"` 生效。

#### 推送 DNS 服务器

``` shell
# 推送 DNS 服务器
push "dhcp-option DNS 192.168.1.1"
```

#### 推送解析

``` shell
# 1. 精准匹配单个完整域名
push "dhcp-option DOMAIN abc.xiaoyekanren.com"

# 2. 匹配根域名及所有子域名
# 指定根域名，隐式支持所有子域名
push "dhcp-option DNSDOMAIN xiaoyekanren.com"

# 3. 配置域名搜索后缀
# DOMAIN-SEARCH 为客户端短域名访问自动补全后缀，不影响 DNS 服务器选择。
# 配置单个搜索后缀
push "dhcp-option DOMAIN-SEARCH xiaoyekanren.com"
# 配置多个搜索后缀，按顺序生效
push "dhcp-option DOMAIN-SEARCH timecho.com"
push "dhcp-option DOMAIN-SEARCH infra.timecho.com"
```

## 2. 疑难杂症

### 1. Ubuntu 客户端 DNS 解析问题

#### 问题现象

Ubuntu 使用 systemd-resolved 管理 DNS，`/etc/resolv.conf` 内容如下：

```
nameserver 127.0.0.53
options edns0 trust-ad
search qh3.cloud.nelbds.org.cn
```

OpenVPN 服务端推送了域名配置，但客户端 ping 域名时仍被 search 后缀干扰，解析到错误IP。

#### 解决方案

1. 安装依赖

``` shell
sudo apt update
sudo apt install -y openvpn-systemd-resolved resolvconf
```

2. 在客户端配置文件中添加

``` shell
# 启用 systemd-resolved 联动
script-security 2
up /etc/openvpn/update-systemd-resolved
down /etc/openvpn/update-systemd-resolved
down-pre
