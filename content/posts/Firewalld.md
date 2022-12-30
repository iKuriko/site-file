---
title: "Firewalld"
date: 2022-12-30T14:22:06+08:00
draft: true
tags:
  - CentOS Linux
  - Linux services
  - Network tools
description: Firewalld是基于区域(zone)的防火墙规则管理工具
---

## 放行协议

放行`drop`区域的`ip`协议

```bash
firewall-cmd --permanent --zone=drop --add-protocol=ip
```

## 放行端口

放行`drop`区域的`tcp:20198`端口

```bash
firewall-cmd --permanent --zone=drop --add-port=20198/tcp
```

## 放行服务

放行`drop`区域的`ssh`服务

```bash
firewall-cmd --permanent --zone=drop --add-service=ssh
```

## 地址伪装

开启地址伪装（类似SNAT的地址映射）

```bash
firewall-cmd --permanent --zone=external --add-masquerade
```

富规则写法：

```bash
firewall-cmd --permanent --zone=external --add-rich-rule='rule family=ipv4 source
address=192.168.10.0/24 masquerade'
```

直接规则写法：

```bash
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -s 192.168.10.0/24 -j SNAT --to-source 1.1.1.1
```

<u>注：直接规则和iptables写法基本相同，配置将会写入`/etc/sysconfig/iptables`的配置文件中</u>

查询和关闭地址伪装

```bash
firewall-cmd --permanent --zone=external --query-masquerade 
firewall-cmd --permanent --zone=external --delete-masquerade 
```

## 端口转发

开启端口转发`本机IP:55265`-->`192.168.10.2:22`

```bash
firewalld-cmd --permanent --zone=external --add-forward-port=port=55265:proto=tcp:toport=22:toaddr=192.168.10.2
```

富贵则写法：

```bash
firewall-cmd --permanent --zone=external --add-rich-rule='rule family=ipv4 source address=192.168.66.0/24 forward-port port=55265 protocol=tcp to-port=22 to-addr=192.168.10.2'
```

直接规则写法：

```bash
firewall-cmd --permanent --zone=external --direct --add-rule ipv4 nat PREROUTING 0 -s 192.168.66.0/24 -d 192.168.66.0/24 -p tcp --dport 55265 -j DNAT --to-destination 192.168.10.2:22
```

查询端口转发

```bash
firewall-cmd --list-forward-ports
```

## 全地址转换

直接规则写法：

```bash
firewall-cmd --permanent --zone=external --direct --add-rule ipv4 nat PREROUTING 0 -s 192.168.68.2/32 -d 192.168.68.1/32 -j DNAT --to-destination 192.168.66.2
firewall-cmd --permanent --zone=external --direct --add-rule ipv4 nat POSTROUTING 0 -s 192.168.66.2/32 -d 192.168.68.2/32 -j SNAT --to-source 192.168.68.1
```

## 加载配置

```bash
firewall-cmd --reload
```

## 命令语法

```bash
firewall-cmd <options>
```

```bash
--list-*    查看
--get-*    获取
--add-*    新建
--remove-*    移除
--set-*    设置
--info-*    信息
--change-*    变更
--query-*    查询
```

## 常用命令

切换默认区域

```bash
firewalld-cmd --permanent --set-default-zone=drop
```

将端口加入某个区域

```bash
firewalld-cmd --permanent --zone=drop --change-interface=ens34
```

查看默认区域信息

```bash
firewalld-cmd --list-all
```

查看所有区域信息

```bash
firewalld-cmd --list-all-zones
```

查看单个区域信息

```bash
firewalld-cmd --list-all --zone=public
```

查看放行的端口 | 服务 | 协议

```bash
firewalld-cmd --list-ports | services | protocols
```

查看富规则

```bash
firewalld-cmd --list-rich-rules
```

查看当前所有区域

```bash
firewalld-cmd --get-zones
```

查看某个接口属于哪个区域

```bash
firewalld-cmd --get-zones-of-interface=ens34
```

紧急模式

```bash
# 开启应急模式阻断所有网络连接
firewall-cmd --panic-on
# 关闭应急模式
firewall-cmd--panic-off
# 查看应急模式的状态
firewall-cmd --query-panic
```

# 
