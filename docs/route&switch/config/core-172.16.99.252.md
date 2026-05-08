# 核心交换机配置 (172.16.99.252)

```
#
 version 7.1.070, Release 7743P05
#
 sysname 252-hexin
#
 telnet server enable
#
 fabric multicast-forwarding mode standard
 multicast forwarding-mode default
#
 dhcp enable
 dhcp server forbidden-ip 172.16.99.200 172.16.99.254
#
 dns proxy enable
 dns server <REDACTED>
 dns server <REDACTED>
 dns server <REDACTED>
 ip host xxxx.xxxxxxx.xxx xxx.xx.xx.xx
 ip host xx.xxxxxxx.xxx xxx.xx.xx.xx
 ip host xxxxxx.xxxxxxx.xxx xxx.xx.xx.xx
 ip host xxxx.xxxxx.xxxxxxx.xxx xxx.xx.xx.xxx
 ip host xxx.xxxxxxx.xxx xxx.xx.xx.xxx
#
 system-working-mode standard
 password-recovery enable
#
vlan 1
#
vlan 100
 description guanli
#
vlan 120
 description youxian
#
vlan 130
 description wuxian
#
vlan 140
 description wuxian-guest
#
 stp global enable
#
dhcp server ip-pool 100
 gateway-list 172.16.99.252
 network 172.16.99.0 mask 255.255.255.0
 dns-list 172.16.99.252
 static-bind ip-address 172.16.99.221 mask 255.255.0.0 hardware-address xxxx-xxxx-xxxx
 static-bind ip-address 172.16.99.231 mask 255.255.0.0 hardware-address xxxx-xxxx-xxxx
 static-bind ip-address 172.16.99.234 mask 255.255.0.0 hardware-address xxxx-xxxx-xxxx
 static-bind ip-address 172.16.99.235 mask 255.255.0.0 hardware-address xxxx-xxxx-xxxx
 static-bind ip-address 172.16.99.241 mask 255.255.0.0 hardware-address xxxx-xxxx-xxxx
 static-bind ip-address 172.16.99.242 mask 255.255.0.0 hardware-address xxxx-xxxx-xxxx
#
dhcp server ip-pool 120
 gateway-list 192.168.99.254
 network 192.168.99.0 mask 255.255.255.0
 dns-list 172.16.99.252
 static-bind ip-address 192.168.99.11 mask 255.255.255.0 hardware-address xxxx-xxxx-xxxx
 static-bind ip-address 192.168.99.12 mask 255.255.255.0 hardware-address xxxx-xxxx-xxxx
 static-bind ip-address 192.168.99.13 mask 255.255.255.0 hardware-address xxxx-xxxx-xxxx
 static-bind ip-address 192.168.99.14 mask 255.255.255.0 hardware-address xxxx-xxxx-xxxx
#
dhcp server ip-pool 130
 gateway-list 192.168.103.254
 network 192.168.100.0 mask 255.255.252.0
 dns-list 172.16.99.252
 static-bind ip-address 192.168.100.80 mask 255.255.0.0 hardware-address xxxx-xxxx-xxxx
#
dhcp server ip-pool 140
 gateway-list 192.168.140.254
 network 192.168.140.0 mask 255.255.255.0
 dns-list 172.16.99.252
#
interface NULL0
#
interface Vlan-interface1
 ip address dhcp-alloc
 dhcp client identifier ascii xxxxxxxxxxxx-VLAN0001
#
interface Vlan-interface100
 description guanli
 ip address 172.16.99.252 255.255.255.0
#
interface Vlan-interface120
 ip address 192.168.99.254 255.255.255.0
#
interface Vlan-interface130
 ip address 192.168.103.254 255.255.252.0
#
interface Vlan-interface140
 ip address 192.168.140.254 255.255.255.0
#
interface GigabitEthernet0/0/1
 port link-mode bridge
 description to-luxiangji
 port access vlan 100
#
interface GigabitEthernet0/0/2
 port link-mode bridge
 description to-ac
 port link-type trunk
 undo port trunk permit vlan 1
 port trunk permit vlan 2 to 4094
#
interface GigabitEthernet0/0/3
 port link-mode bridge
 description to-camera-poe
 port access vlan 100
#
interface GigabitEthernet0/0/4
 port link-mode bridge
 description to-jigui-2
 port access vlan 100
#
interface Ten-GigabitEthernet0/0/17
 port link-mode bridge
 description to-sw-poe
 port link-type trunk
 undo port trunk permit vlan 1
 port trunk permit vlan 2 to 4094
#
interface Ten-GigabitEthernet0/0/18
 port link-mode bridge
 description to-sw1
 port link-type trunk
 undo port trunk permit vlan 1
 port trunk permit vlan 2 to 4094
#
interface Ten-GigabitEthernet0/0/19
 port link-mode bridge
 description to-sw2
 port link-type trunk
 undo port trunk permit vlan 1
 port trunk permit vlan 2 to 4094
#
 scheduler logfile size 16
#
line class console
 user-role network-admin
#
line class vty
 user-role network-operator
#
line con 0
 user-role network-admin
#
line vty 0 4
 authentication-mode scheme
 user-role network-operator
#
line vty 5 63
 user-role network-operator
#
 ip route-static 0.0.0.0 0 172.16.99.254
#
domain system
#
 domain default enable system
#
local-user xxxxxxx class manage
 password hash <REDACTED>
 service-type telnet http https
 authorization-attribute user-role network-admin
 authorization-attribute user-role network-operator
#
 ip http enable
 ip https enable
#
return
```