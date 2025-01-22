---
title: "OpenvSwitch"
date: 2022-10-14T15:50:53+08:00
draft: true
tags:
  - OpenvSwitch
description: 虚拟交换机软件,主要用于虚拟机VM环境,支持Xen/XenServer,KVM以及VirtualBox多种虚拟化技术
---



## 软件包下载

官网自选下载：[http://www.openvswitch.org/](http://www.openvswitch.org/)

## 软件包安装

```bash
tar -zxf openvswitch-2.16.2-1.el8.x86_64-centos-8.5.tar.gz
```

```bash
cd openvswitch-2.16.2-1.el8.x86_64-centos-8.5
```

```bash
yum localinstall -y openvswitch-2.16.2-1.el8.x86_64.rpm openvswitch-devel-2.16.2-1.el8.x86_64.rpm openvswitch-kmod-2.16.2-1.el8.x86_64.rpm 
```

```bash
systemctl start openvswitch.service ;systemctl enable openvswitch.service
```

```bash
ovs-vsctl -V
ovs-vsctl (Open vSwitch) 2.16.2
DB Schema 8.3.0
```






## 常用命令

添加|删除 网桥

```bash
ovs-vsctl add-br | del-br <br-name>
```

添加 | 删除 接口

```bash
ovs-vsctl add-port | del-port <br-name> <port-name>
```

查看接口加入的网桥

```bash
ovs-vsctl port-to-br <port-name> 
```

查看单个桥中的接口

```bash
ovs-vsctl list-ports <br-name> 
```

查看单个桥的详细信息

```bash
ovs-ofctl show <br-name> 
```

开启 | 关闭 STP

```bash
ovs-vsctl set bridge <br-name> stp_enable=true | false
```

修改MTU

```bash
ovs-vsctl set Interface <br-name> mtu_request=1300
```







查看所有的桥和接口

```bash
ovs-vsctl show
```

只查看所有桥名

```bash
ovs-vsctl list-br
```



查看软件版本

```bash
ovs-vsctl -V
```

查看接口状态（光口SFP模块信息）

```bash
ovs-invctl enable-sfp true
```

```bash
ovs-invctl show-sfp-qsfp
```



添加默认流表

```bash
ovs-ofctl add-flow <br-name> priority=1,actions=NORMAL
```

查看流表



```bash
ovs-ofctl dump-flows <br-name>
```
