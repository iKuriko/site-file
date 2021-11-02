---
title: "Nmcli"
date: 2021-11-02T14:37:25+08:00
draft: true
---



## nmcli 常用命令



### 查看网络连接

**列出所有网络设备**

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



### 添加网络连接

**新建网络连接**

```
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







### 修改网络连接

**添加IPv4地址**

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

添加IP地址，网关和DNS，IP获取方式设置为手动

```bash
nmcli connection modify ens33 ipv4.method manual ipv4.addresses 192.168.0.1/24 ipv4.gateway 192.168.0.254 ipv4.dns 8.8.8.8
```

添加第二个DNS

```bash
nmcli connection modify ens33 +ipv4.dns 8.8.8.8 备用DNS
```

**添加静态路由**

```bash
nmcli connection modify ens33 +ipv4.routes "192.168.2.0/24 192.168.1.233"
```



### 控制网络连接

**启用设备**

```bash
nmcli connection up ens33
```

关闭设备

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





### 添加虚拟设备

**新建网桥**

```bash
nmcli connection add type bridge ifname Bridge con-name Bridge
```

配置网桥ipv4地址禁用，ipv6地址禁用

```bash
nmcli connection modify Bridge ipv4.method disable ipv6.method disable
```

启用设备，生效配置

```bash
nmcli connection up Bridge
```

**配置gretap**

```bash
nmcli connection add type ip-tunnel mode gretap remote 172.16.100.100 local 172.16.100.1 ifname gretap01  con-name gretap01 
```

配置网卡ipv4地址手动，ipv6地址禁用

```bash
nmcli connection modify gretap01 ipv4.method manual ipv6.method disable
```

启用设备，生效配置

```bash
nmcli connection up gretap01
```



**配置GRE**

```
nmcli connection add type ip-tunnel mode gre con-name gre01 ifname gre01 remote 172.16.200.200 local 172.16.100.1 ipv4.addresses 192.168.0.1/30
```

配置网卡ipv4地址手动，ipv6地址禁用

```bash
nmcli connection modify gre01 ipv4.method manual ipv6.method disable
```

启用设备，生效配置

```bash
nmcli connection up gre01
```



**配置vlan子接口**

```bash
nmcli connection add type vlan con-name ens33.1 ifname ens33.1 id 1 master Bridge dev ens33
```

启用设备，生效配置

```bash
nmcli connection up ens33.1
```





