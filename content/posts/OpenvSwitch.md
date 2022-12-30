---
title: "OpenvSwitch"
date: 2022-10-14T15:50:53+08:00
draft: true
tags:
  - CentOS Linux
  - Network tools
  - Linux services
description: 虚拟交换机的软件,主要用于虚拟机VM环境,支持Xen/XenServer,KVM以及VirtualBox多种虚拟化技术
---

常用命令

```bash
ovs-vsctl -V
```



```bash
ovs-vsctl show
```



```bash
ovs-vsctl list-br
```



```bash
ovs-vsctl add-br | del-br <br-name>
```



```bash
ovs-vsctl add-port | del-port <br-name> <port-name>
```



```bash
ovs-vsctl list-ports <br-name> 
```



```bash
ovs-vsctl port-to-br <br-name> 
```



```bash
ovs-ofctl show <br-name> 
```



查看接口收光

```
ovs-invctl enable-sfp true
```

```
ovs-invctl show-sfp-qsfp
```
