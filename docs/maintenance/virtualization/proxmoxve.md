---
category: 系统
---

# ProxmoxVE 硬盘直通

通过 RDM 磁盘直通方式，将物理硬盘独占分配给虚拟机。

参考：https://foxi.buduanwang.vip/virtualization/1754.html/

## 1. 列出硬盘列表

``` shell
ls -la /dev/disk/by-id/|grep -v dm|grep -v lvm|grep -v part
# scsi-3600062b20xxxxxx0304e83cc04cde739 -> ../../sda
# scsi-3600062b20xxxxxx0304e909204e398eb -> ../../sdb
# scsi-3600062b20xxxxxx0304e92030504e296 -> ../../sdc
# scsi-3600062b20xxxxxx0304e9237081d3f41 -> ../../sdd
# scsi-3600062b20xxxxxx0304e925c0a5c3481 -> ../../sde
```

## 2. 将硬盘直通到虚拟机

``` shell
# qm set <vmid> --scsiX /dev/disk/by-id/xxxxxxx
qm set 101 --scsi0 /dev/disk/by-id/scsi-3600062b20xxxxxx0304e909204e398eb
qm set 102 --scsi0 /dev/disk/by-id/scsi-3600062b20xxxxxx0304e92030504e296
qm set 103 --scsi0 /dev/disk/by-id/scsi-3600062b20xxxxxx0304e9237081d3f41
qm set 104 --scsi0 /dev/disk/by-id/scsi-3600062b20xxxxxx0304e925c0a5c3481
