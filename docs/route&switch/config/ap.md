# AP 配置

```
<1496-2de9-7280>dis cu
#
 version 7.1.064, Release 2452P06
#
 sysname 1496-2de9-7280
#
 telnet server enable
#
 lldp global enable
#
 password-recovery enable
#
vlan 1
#
vlan 130
#
vlan 140
#
nqa template icmp apmgr-aptoac-icmp
 data-size 1500
 frequency 1000
 probe timeout 1000
 reaction trigger per-probe
#
interface NULL0
#
interface Vlan-interface1
 ip address dhcp-alloc
#
interface GigabitEthernet1/0/1
#
interface WLAN-Radio1/0/1
#
interface WLAN-Radio1/0/2
#
interface WLAN-Radio1/0/3
#
interface Smartrate-Ethernet1/0/1
 port link-type trunk
 port trunk permit vlan 1 130
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
 user-role network-admin
 set authentication password hash <REDACTED>
#
line vty 5 63
 user-role network-operator
#
undo gratuitous-arp-learning enable
#
domain system
#
 domain default enable system
#
user-group system
#
return
```