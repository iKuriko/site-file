---
title: "CentOS8 时间同步"
date: 2021-12-24T14:28:17+08:00
draft: true
description: 
---



> CentOS 系统安装过程中，会有选择时区的步骤。如果时区设置有误，可能导致系统时间与本地时间不同步。使得一些应用程序的时间戳发生错乱，出现各种报错。



设置时区

```bash
timedatectl set-timezone Asia/Shanghai
```

```bash
timedatectl set-ntp true
```

​    or

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```



> 要与任意设备进行同步时，在 CentOS 8 中需要安装 chronyd 服务

安装服务

```bash
yum -y install chronyd
```

编辑配置

```bash
vim /etc/chrony.conf
+ server ntp.aliyun.com iburst
```

重启服务

```bash
systemctl restart chronyd
```

立即同步

```bash
chronyc sources -v
```

 
