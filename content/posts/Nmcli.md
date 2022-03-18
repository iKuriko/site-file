---
title: "Nmcli 命令使用"
date: 2021-11-02T14:37:25+08:00
draft: true
tags:
  - CentOS Linux
  - NetworkManager
description: Nmcli 由 网络管理器 NetworkManager 软件包提供，是新一代的网络管理工具
---



## 常用命令

### 查看

列出所有网络设备

```bash
nmcli connection show
```

查看所有活动的连接

```bash
nmcli connection show --active
```

查看某个网络设备的详细信息

```bash
nmcli connection show ens33
```

查看网络设备状态

```bash
nmcli device status
```

### 添加

新建网络连接

```bash
nmcli connection add type ethernet con-name ens33 ifname ens33
```

删除网络连接

```bash
nmcli connection delete ens33
```

重新加载配置文件

```bash
nmcli connection reload
```

### 修改

修改IPv4地址的获取方式为手动

```bash
nmcli connection modify ens34 ipv4.method manual
```

添加第一个IPv4地址

```bash
nmcli con modify ens34 ipv4.addresses 192.168.1.1/24
```

添加第二个IPv4地址

```bash
nmcli con modify ens34 +ipv4.addresses 192.168.2.1/24
```

删除IPv4地址

```bash
nmcli con modify ens34 -ipv4.addresses 192.168.2.1/24
```

添加IP地址，网关和DNS

```bash
nmcli connection modify ens34 ipv4.gateway 192.168.0.254 ipv4.dns 8.8.8.8
```

添加第二个DNS(备用DNS)

```bash
nmcli connection modify ens34 +ipv4.dns 8.8.8.8
```

添加静态路由

```bash
nmcli connection modify ens34 +ipv4.routes "192.168.2.0/24 192.168.1.233"
```

设置端口自动连接

```bash
nmcli connection modify ens34 autoconnect yes
```

### 控制

启用网络设备

```bash
nmcli connection up ens33
```

关闭网络设备

```bash
nmcli connection down ens33
```

断开物理设备

```bash
nmcli device disconnec ens33
```

连接物理设备


```bash
nmcli device connect ens33
```

## 创建虚拟设备

### Linux-br

创建名为 Brdige 的 Linux 网桥

```bash
nmcli connection add type bridge ifname Bridge con-name Bridge ipv4.method disable ipv6.method disable
```

### gretap

创建名为 gretap01 的 gretap 二层隧道，加入 Brdige 网桥

```bash
nmcli connection add type ip-tunnel mode gretap con-name gretap01 ifname gretap01 ip-tunnel.ttl 255 ip-tunnel.input-key 1 ip-tunnel.output-key 1 local 192.168.2.164 remote 192.168.2.91 ipv4.method disabled ipv6.method disabled master Bridge
```

nmcli 参数详解：

add：新建连接

type：连接类型

mode：ip-tunnel的隧道模式

con-name：网络连接名称

ifname：设备接口名称

ip-tunnel.ttl：ttl生命周期

ip-tunnel.input-key：gre隧道的key值（input）

ip-tunnel.output-key：gre隧道的key值（output）

local：本地的IP地址

remote：远程的IP地址

autoconnect：自动连接  

ipv4.method：IPv4 配置方法

ipv6.method：IPv6 配置方法

master：隧道加入的网桥（按需添加）

### GRE

创建名为 gre01 的 GRE 三层隧道

```bash
nmcli connection add type ip-tunnel mode gre con-name gre01 ifname gre01 remote 192.168.2.91 local 192.168.2.164 ipv4.addresses 172.233.1.2/24 ip-tunnel.input-key 1 ip-tunnel.output-key 1 ip-tunnel.ttl 255 ipv4.method manual ipv6.method disable
```

### vlan

创建基于 ens33 网卡名为 ens33.1 的 vlan 子接口，加入 Bridge 网桥

```bash
nmcli connection add type vlan con-name ens33.1 ifname ens33.1 id 1 master Bridge dev ens33
```

### vxlan

**vxlan创建命令（三层）**

创建名为 vxlan1 的三层 vxlan

```bash
nmcli connection add type vxlan id 100 remote 192.168.2.91 ipv4.addresses 172.233.233.1/24 ipv4.method manual ipv6.method disabled ifname vxlan1 connection.id vxlan1 vxlan.parent ens32
```

**vxlan创建命令（二层）**

创建名为 vxlan10 的二层 vxlan ，加入 Brdige 网桥

```bash
nmcli connection add type vxlan id 10 remote 192.168.2.91 ipv4.method disabled ipv6.method disabled ifname vxlan10 connection.id vxlan10 vxlan.parent ens32 master Bridge
```





