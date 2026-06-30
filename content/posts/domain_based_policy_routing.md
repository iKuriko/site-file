---
title: "Domain-based Policy Routing"
date: 2026-06-30T14:35:30+08:00
draft: true
tags:
  - Tools
description: 基于域名的策略路由是将 dnsmasq（DNS 解析与 ipset 填充）、ipset（IP 集合管理）、iptables（流量标记）和 iproute2（策略路由）这四个成熟的组件创造性地组合在一起的经典范例
---



**如何让防火墙（iptables）和路由策略（iproute），能识别并处理一个不断变化的 IP 地址列表** 

先模拟一个使用场景

1、有几个固定的目标IP需要从`WAN1`口出去访问

2、有一些目标域名需要从`WAN2`口出去访问

3、还有一些目标域名需要指定的`dns server`来解析并且指定任意出口访问



## IPv4使用示例

### 配置ipset 

创建目标静态ip列表

``` bash
ipset create static_ip hash:net
```

创建dnsmasq域名解析返回的目标动态ip列表

``` bash
ipset create dynamic_ip hash:net timeout 1800
```



### 配置dnsmasq

创建dnsmasq目录
``` bash
mkdir -p /sdn/shell/dnsmasq
```
编辑dnsmasq配置文件
``` bash
vim /sdn/shell/dnsmasq/dnsmasq.conf

ipset=/./dynamic_ip
```
启动dnsmasq进程
``` bash
dnsmasq -C /sdn/shell/dnsmasq/dnsmasq.conf
```



### 配置防火墙 

从`lan`口进来的数据包，目的地址匹配`static_ip`表的打标签`0xa1001`
``` bash
iptables -t mangle -A PREROUTING -i <input_port> -m set --match-set static_ip dst -j MARK --set-xmark 0xa1001/0xffffffff
iptables -t mangle -A PREROUTING -i <input_port> -m set --match-set static_ip dst -j RETURN
```
从`lan`口进来的数据包，目的地址匹配dnsmasq域名解析返回的`dynamic_ip`表的打标签`0xa1002`
``` bash
iptables -t mangle -A PREROUTING -i <input_port> -m set --match-set dynamic_ip dst -j MARK --set-xmark 0xa1002/0xffffffff
iptables -t mangle -A PREROUTING -i <input_port> -m set --match-set dynamic_ip dst -j RETURN
```
从`wan1`口出去数据包SNAT
``` bash
iptables -t nat -A POSTROUTING -o <output_port1> -j MASQUERADE
```
从`wan2`口出去数据包SNAT
``` bash
iptables -t nat -A POSTROUTING -o <output_port2> -j MASQUERADE
```
劫持DNS到本地IP地址
``` bash
iptables -t nat -A PREROUTING -p tcp -m tcp --dport 53 -j DNAT --to-destination <local_ip>
iptables -t nat -A PREROUTING -p udp -m udp --dport 53 -j DNAT --to-destination <local_ip>
```

### 配置策略路由

标签号是`0xa1001`的数据包查看`1001`路由表
``` bash
ip ru add pref 20000 from all fwmark 0xa1001 lookup 1001
```
给`1001`路由表添加default路由指向`wan1`网关
``` bash
ip ro add tab 1001 default via <wan1 gateway>
```
标签号是`0xa1002`的数据包查看`1002`路由表
``` bash
ip ru add pref 20000 from all fwmark 0xa1002 lookup 1002
```
给`1002`路由表添加default路由指向`wan2`网关
``` bash
ip ro add tab 1002 default via <wan2 gateway>
```
### 添加静态目标IP至ipset


``` bash
ipset add static_ip 223.5.5.5/32
```


如果有一整个IP表，可以使用循环添加
``` bash
for i in `cat static_ip.txt` 
do
ipset add static_ip $i 
done
```



### 非本地解析

如果某些域名想要使用非本地dns解析，可以添加到dnsmasq，然后防火墙放行目标`dns server`

``` bash
vim /sdn/shell/dnsmasq/dnsmasq.conf

ipset=/./dynamic_ip
server=/www.ccb.com/223.5.5.5
```
先杀掉然后重启dnsmasq进程
``` bash
ps -elf | grep dnsmasq | grep -v grep | awk '{print $4}' | xargs kill -9 
```
``` bash
dnsmasq -C /sdn/shell/dnsmasq/dnsmasq.conf
```
配置防火墙目标`dns server`指定`wan2`出口
``` bash
iptables -t mangle -A OUTPUT -d 223.5.5.5 -j MARK --set-xmark 0xa1002/0xffffffff
```



## IPv6使用示例

### 配置ipset

创建目标静态ipv6列表

``` bash
ipset create static_ipv6 hash:net family inet6
```
创建dnsmasq域名解析返回的目标动态ipv6列表
``` bash
ipset create dnsmasq_ipv6 hash:net timeout 1800 family inet6
```
### 配置dnsmasq

创建dnsmasq目录

``` bash
mkdir -p /sdn/shell/dnsmasq
```
编辑dnsmasq配置文件
``` bash
vim /sdn/shell/dnsmasq/dnsmasq.conf

ipset=/./dynamic_ipv6
```
启动dnsmasq进程
``` bash
dnsmasq -C /sdn/shell/dnsmasq/dnsmasq.conf
```
### 配置防火墙

从`lan`口进来的数据包，目的地址匹配`static_ipv6`表的打标签`0xa1003`

``` bash
ip6tables -t mangle -A PREROUTING -i enp1s0 -m set --match-set static_ipv6 dst -j MARK --set-xmark 0xa1003/0xffffffff
ip6tables -t mangle -A PREROUTING -i enp1s0 -m set --match-set static_ipv6 dst -j RETURN
```
从`lan`口进来的数据包，目的地址匹配`dnsmasq`域名解析返回的`dynamic_ipv6`表的打标签`0xa1004`
``` bash
ip6tables -t mangle -A PREROUTING -i enp1s0 -m set --match-set dynamic_ipv6 dst -j MARK --set-xmark 0xa1004/0xffffffff
ip6tables -t mangle -A PREROUTING -i enp1s0 -m set --match-set dynamic_ipv6 dst -j RETURN
```
从`wan1`口出去数据包的SNAT
``` bash
ip6tables -t nat -A POSTROUTING -o wan1 -j MASQUERADE
```
从`wan2`口出去数据包SNAT
``` bash
ip6tables -t nat -A POSTROUTING -o wan2 -j MASQUERADE
```
### 配置策略路由

标签号是`0xa1003`的数据包查看`1003`路由表

``` bash
ip -6 ru add pref 20000 from all fwmark 0xa1003 lookup 1003
```
给`1003`路由表添加default路由指向`wan1`网关
``` bash
ip -6 ro add tab 1003 default via <wan1 gateway>
```
标签号是`0xa1004`的数据包查看1004路由表
``` bash
ip -6 ru add pref 20000 from all fwmark 0xa1004 lookup 1004
```
给`1004`路由表添加default路由指向`wan2`网关
``` bash
ip -6 ro add tab 1004 default via <wan2 gateway>
```
### 添加静态目标IPv6至ipset

```bash
ipset add static_ipv6 2002::1/128
```

如果有一整个IPv6表，可以使用循环添加

``` bash
for i in `cat static_ipv6.txt` 
do
	ipset add static_ipv6 $i 
done
```
### 非本地解析

如果某些域名想要使用非本地dns解析，可以添加到dnsmasq，然后防火墙放行目标`dns server`

``` bash
vim /sdn/shell/dnsmasq/dnsmasq.conf

ipset=/./dynamic_ipv6
server=/www.ccb.com/223.5.5.5
```
先杀掉然后重启dnsmasq进程
``` bash
ps -elf | grep dnsmasq | grep -v grep | awk '{print $4}' | xargs kill -9 
```
``` bash
dnsmasq -C /sdn/shell/dnsmasq/dnsmasq.conf
```
配置防火墙目标`dns server`指定`wan2`出口
``` bash
ip6tables -t mangle -A OUTPUT -d 223.5.5.5 -j MARK --set-xmark 0xa1004/0xffffffff
```