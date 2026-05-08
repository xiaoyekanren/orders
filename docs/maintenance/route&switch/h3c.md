# H3C 常用命令

## 通用

``` shell
# 显示 IP 地址
dis arp

# 显示物理端口启用状态
dis int bri

# 显示 MAC 地址
dis mac-address
dis mac-address interface GigabitEthernet1/0/5
```

## AC 控制器

``` shell
# 列出全部 AP
display wlan ap all

# 开启 AP 远程登录权限（在 AC 上执行）
sys
probe
wlan ap-execute all exec-console enable
```

## AP 默认密码

```
xxxxxxxxxx
```

## 增加 SSID 不生效

确保以下步骤均执行：

1. **AC**
   - 新建 SSID
   - 增加 VLAN
   - 确保下发文件：apcfg.txt
   - 修改 apcfg，增加一行放通 VLAN
   - 重启全部 AP（或手动增加该命令）

2. **核心交换机**
   - 增加 VLAN
   - 增加 VLAN interface
   - 增加 DHCP server

3. **POE 交换机**
   - 增加 VLAN

4. **途经的全部交换机**
   - 增加 VLAN

## 恢复出厂设置

``` shell
restore factory-default
```

## 切换到瘦模式

``` shell
probe
ap-mode fit
```