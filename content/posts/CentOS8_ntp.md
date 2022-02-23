---
title: "CentOS8 时间同步"
date: 2021-12-24T14:28:17+08:00
draft: true
description: CentOS 系统安装过程中，会有选择时区的步骤。如果时区设置有误，可能导致系统时间与本地时间不同步。使得一些应用程序的时间戳发生错乱，出现各种报错。
---



## 设置系统时区

查看时区

```bash
timedatectl
```

选择时区

```bash
timedatectl set-timezone Asia/Shanghai
```

手动设置时间

```bash
timedatectl set-time "2022-02-22 09:00:00"
date -s "2022-02-22 09:00:00"
```

创建软连接设置本地时间

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```



## Chronyd服务



要与任意主机进行同步时

- "CentOS 8"需要安装 chronyd 服务
- "CentOS 7"执行`ntpdate ntp.aliyun.com`即可同步



安装Chronyd服务

```bash
yum -y install chronyd
```

添加需要同步的主机

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

 开启时间自动同步

```bash
timedatectl set-ntp true
```

