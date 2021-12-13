---
title: "Linux 热添加硬盘"
date: 2021-11-01T17:41:41+08:00
draft: true
---

Linux服务器运行时，扫描新插入的硬盘 

此环境为VMware虚拟机，服务器真机同样适用  

  

扫描所有磁盘  

```bash
for i in /sys/class/scsi_host/host*/scan;do echo "- - -" >$i;done
```

 命令解释

'- - -'代表channel，target和LUN编号。以上命令会导致host下所有channel，target以及可见LUN被扫描  
    

or  




添加磁盘：

```bash
echo “scsi add-single-device 1 2 3 4” >/proc/scsi/scsi
```

移除硬盘：

```bash
echo “scsi remove-single-device 1 2 3 4” > /proc/scsi/scsi
```



命令行中的 1 2 3 4 需要自行修改成相应的参数：

- 1 : SCSI HBA ID  主机适配器标识，第一个适配器为零(0)
- 2 : SCSI Channel  主机适配器上的 SCSI 通道，第一个通道为零(0)
- 3 : SCSI ID  设备的 SCSI 标识
- 4 : LUN ID   LUN 号，第一个 LUN 为零(0)

查看scsi磁盘的命令

```bash
lsscsi

[1:0:0:0]  cd/dvd NECVMWar VMware IDE CDR10 1.00 /dev/sr0 
[2:0:0:0]  disk  VMware, VMware Virtual S 1.0  /dev/sda 
[2:0:1:0]  disk  VMware, VMware Virtual S 1.0  /dev/sdb 
```

```bash
cat /proc/scsi/scsi 

Attached devices:
Host: scsi2 Channel: 00 Id: 00 Lun: 00
 Vendor: VMware, Model: VMware Virtual S Rev: 1.0 
 Type:  Direct-Access          ANSI SCSI revision: 02
Host: scsi1 Channel: 00 Id: 00 Lun: 00
 Vendor: NECVMWar Model: VMware IDE CDR10 Rev: 1.00
 Type:  CD-ROM              ANSI SCSI revision: 05
Host: scsi2 Channel: 00 Id: 01 Lun: 00
 Vendor: VMware, Model: VMware Virtual S Rev: 1.0 
 Type:  Direct-Access          ANSI SCSI revision: 02
```

未验证的方法

~~\#使用rpm包sg3_utils 中的rescan-scsi-bus.sh脚本重新扫描~~

~~\# /usr/bin/rescan-scsi-bus.sh~~

