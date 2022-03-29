---
title: "Iptables"
date: 2022-03-28T11:23:42+08:00
draft: true
tags:
  - iptables
  - CentOS Linux
description: netfilter/iptables 是 Linux 内核集成的IP信息包过滤系统，有利于在 Linux 系统上更好的控制IP信息包过滤和防火墙配置
---



## iptables 简介



iptables 是Linux内核集成的IP信息包过滤系统，有利于在 Linux 系统上更好的控制IP信息包过滤和防火墙[^1]配置。

iptables 防火墙在做数据包的过滤和转发时，有一套自己遵循的规则，这些规则存储在专用的数据包过滤表中，这些表集成在 Linux 的内核中。

在数据包过滤时，这些规则被分组存放在所谓的链（chain），而 netfilter/iptables IP数据包过滤系统是一款强大的工具，可以用于添加、编辑和删除防火墙的规则。



## iptables 组成

虽然 netfilter/iptables IP信息包过滤系统被称为单个实体，但它实际上由 netfilter 和 iptables 两个组件构成。

1. netfilter 组件是存储过滤规则的表，也称为内核空间（kernelspace），是内核的一部分，由一些信息包过滤表组成，这些表包含内核用来控制信息包过滤处理的规则集。
2. Iptables 组件是一种工具，也称为用户空间（userspace），它使得新增、修改和删除信息包过滤表（netfilter）中的规则变得简单容易。

 **组成总结**

1. netfilter：Linux内核模块，提供防火墙功能，但用户不可干预；内核态
2. iptables：用于编写防火墙规则的工具（类似BIOS和CMOS之间的关系），修改的操作都会写入到netfilter中；用户态



## iptables 四表五链

| 表     | 链         |             |             |           |        |
| :----- | :--------- | ----------- | ----------- | --------- | ------ |
| raw    | PREROUTING | OUTPUT      |             |           |        |
| mangle | INPUT      | PREROUTING  | POSTROUTING | FORWARD   | OUTPUT |
| nat    | PREROUTING | POSTROUTING | OUTPUT      | INPUT[^2] |        |
| filter | INPUT      | OUTPUT      | FORWARD     |           |        |



**四表：每个表有不同的功能** *\*表：按照功能进行分类。*

| 表名   | 作用       |
| ------ | ---------- |
| raw    | 状态跟踪   |
| mangle | 标记数据包 |
| nat    | 地址转换   |
| filter | 地址过滤   |

**五链：针对不同的时机 **\**链：按照时机进行分类。*

| 链名        | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| INPUT       | 过滤入站的数据包，过滤指定的包到达本地服务器                 |
| OUTPUT      | 过滤出站数据包，限制本地服务器发出指定的包，一般不做控制，全部放行 |
| FORWARD     | 转发数据包，一般用于网关型防火墙，实现数据包的转发           |
| PREROUTING  | 路由前的数据包，用于在网关型防火墙下，实现DNAT，将内网的主机发布到公网，让公网用户访问 |
| POSTROUTING | 路由后的数据包，用于在网关型防火墙下，实现SNAT，用于内网主机共享独立公网IP，实现上网需求 |







## iptables 匹配顺序



**表匹配的顺序**

| raw  | mangle | nat  | filter |
| ---- | ------ | ---- | ------ |



**链匹配的顺序**

| 动作 | 链名       |             |             |
| :--- | :--------- | ----------- | ----------- |
| 入站 | PREROUTING | INPUT       |             |
| 出站 | OUTPUT     | POSTROUTING |             |
| 转发 | PREROUTING | FORWARD     | POSTROUTING |
| 主机 | INPUT      |             |             |
| 网关 | PREROUTING | POSTROUTING | FORWARD     |





## iptables 安装



CentOS7以上的版本已不自带iptables-services的服务本体

```bash
yum -y install iptables iptables-services
```

禁用firewalld

```bash
systemctl stop firewalld
systemctl disable firewalld
```

关闭SELINUX

```bash
setenforce 0    #临时关闭
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config    #永久关闭，需重启
```



如果要把安装了iptables的设备当作网关防火墙，需要开启路由转发功能

1.修改配置文件

```bash
vim /etc/sysctl.conf
+ net.ipv4.ip_forward = 1    #开启ipv4路由转发
```

```bash
sysctl -p    #立即生效
```

2.或者直接修改内核参数

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward    #跟上面的方法效果相同
```



## iptables 规则编写

iptables防火墙就是规则策略的编写、它规定了数据包在什么时机完成什么事。

### 新建规则

```bash
iptables [-t 表名] [选项] [链名] [匹配条件] [-j 控制类型]
```

选项：

```bash
-A ：链中最后增加一条
-I：链中最开始增加一条
-D：删除指定链
-L：列表显示所有规则
-n：以数字显示端口、IP等
-v：详细信息显示
-P：指定默认规则
--line-numbers：显示规则的序号
```

匹配条件1、通用匹配：

```bash
-p：协议(tcp、udp、icmp)
-s：源地址
-d：目标地址
-i：入口网卡
-o：出口网卡
```

匹配条件2、隐含匹配(需配合-p选项)：

```bash
--dport：目标端口
--sport：源端口
--icmp-type：icmp协议类型；0(回显)、3(网络不可达)、8(请求)
--tcp-flages：TCP标记
```

匹配条件3、显式匹配：

```bash
-m multiport --sports：源端口列表
-m multiport --dports：目标端口列表
-m iprange --src-range：IP地址范围
-m mac --mac-source：MAC地址
-m state --state：连接状态；NEW(DROP)、ESTABLISHED(ACCEPT)、RELATED(ACCEPT)
```

控制类型：

```bash
ACCEPT：允许
REJECT：拒绝
DROP：丢弃
LOG：日志
```





### 删除规则

- **-F** 是清空指定某个 chains 内所有的 rule 设定。比方 iptables -F -t filter，那就是把 filter table 内所有的INPUT/OUTPUT/FORWARD chain 设定的规则都清空。
- **-X** 是删除使用者自订 table 项目，一般使用 iptables -N xxx 新增自订 chain 后，可以使用 iptables -X xxx 删除之。



```bash
iptables -F    #删除所有规则
```

```bash
iptables -t nat -F    #删除nat表中的规则
```



### 查看规则

```bash
iptables-save    #查看当前所有规则
```

```bash
iptables -L -n -v    #查看所有规则
iptables -t nat -L -n -v    #查看nat表的规则
```



### 保存规则

```bash
service iptables save    #保存规则
```

**\* 不要忘记使用<font color=red>`service iptables save` </font>或 <font color=red>`iptables-saves > /etc/sysconfig/iptables`</font>保存规则，否则都为一次性配置，重启服务后消失**



### 规则案例

通用匹配案例：

```bash
vim clean_iptables.sh
#!/bin/bash
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X
iptables -t raw -F
iptables -t raw -X
iptables -t security -F
iptables -t security -X
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -A FORWARD -i eth0 -o veth0 -j ACCEPT    #允许口对口的数据转发，veth0和eth0
iptables -A FORWARD -o eth0 -i veth0 -j ACCEPT
iptables -P OUTPUT ACCEPT
iptables -t filter -I INPUT -s 192.168.12.0/24 -p icmp -j ACCEPT    #允许192.168.12.0网段使用icmp协议访问
iptables -t filter -P INPUT DROP    #设置默认INPUT链为拒绝
```



隐含匹配案例：

```bash
iptables -t filter -I INPUT -p tcp --dport 22 -s 192.168.12.0/24-j ACCEPT    #允许192.168.12.0网段访问本机的22端口
iptalbes -t filter -P INPUT DROP
```



显式匹配案例：

```bash
iptables -t filter -I INPUT -p icmp --icmp-type 0 -j ACCEPT
iptables -t filter -I INPUT -p icmp --icmp-type 3 -j ACCEPT
iptables -t filter -I INPUT -p icmp --icmp-type 8 -j DROP
iptalbes -t filter -P INPUT DROP    #服务器可以ping外部，但外部不可ping服务器
```



### raw表的使用

**RAW表：**

只使用在 PREROUTING 链和 OUTPUT 链上，因为优先级最高，从而可以对收到的数据包在连接跟踪前进行处理。一旦用户使用了 RAW 表，在某个链上，RAW表处理完后，将跳过 NAT 表和 ip_conntrack 的处理，即不再做地址转换和数据包的链接跟踪。



**使用场景：**

RAW表可以应用在那些不需要做nat的情况下，以提高性能。如拥有大量访问的web服务器，可以规定访问80端口不再让iptables做数据包的链接跟踪处理，以提高用户的访问速度。

添加raw表在其他表处理之前，使用`-j NOTRACK`跳过其它表处理。控制类型除了以前的四个还增加了一个`UNTRACKED`

**使用案例：**

添加raw表规则，可以使用 “NOTRACK” target 允许规则，指定80端口的包不进入链接跟踪/NAT子系统，放行“内网–>外网”的Web访问，不做状态跟踪/NAT

 

将目标地址101.66.58.9，目的端口80的数据包，跳过其他表的处理

```bash
iptables -t raw -A PREROUTING -d 101.66.58.9 -p tcp --dport 80 -j NOTRACK
```

将源地址101.66.58.9，源端口80的数据包，跳过其他表的处理

```bash
iptables -t raw -A PREROUTING -s 101.66.58.9 -p tcp --sport 80 -j NOTRACK
```

设置 FORWARD 链的状态为 UNTRACKED（不跟踪）

```bash
iptables -A FORWARD -m state --state UNTRACKED -j ACCEPT
```





### mangle表的使用

标记数据包，可以配合策略路由使用

```bash
iptables -t mangle -A PREROUTING -p tcp -m multiport --dports 80,8080,20,21 -s 192.168.1.0/24 -j MARK --set-mark 1  
```

```bash
ip rule add fwmark 1 pref 1002 table test2 
```



系统**tcpmss**的更改

```bash
iptables -t mangle -A FORWARD -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --set-mss 1300
```



### filter表的使用

允许192.168.12.0网段使用icmp协议访问

```bash
iptables -t filter -I INPUT -s 192.168.12.0/24 -p icmp -j ACCEPT
```

允许192.168.12.0网段访问本机的22端口

```bash
iptables -t filter -I INPUT -p tcp --dport 22 -s 192.168.12.0/24-j ACCEPT
```

已经建立连接的包以及该连接相关的包允许通过

```bash
iptables -t filter -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

允许目的端口是22,80,443的数据包通过

```bash
iptables -t filter -A INPUT -p tcp -m multiport --dports 22,80,443 -j ACCEPT
```

设置默认INPUT链为拒绝

```bash
iptables -t filter -P INPUT DROP
```



## SNAT与DNAT

### DNAT规则

DNAT：目标地址转换，在路由开启前，将数据包的目标地址更换为内网段的IP地址



**PREROUTING（路由前）详解：**

1. 前提：内网主机192.168.1.10/24:80通过地址转换映射到公网地址200.0.0.1/24:80

2. 此时有一个公网的客户端200.0.0.2想要访问我们的内网主机的web服务192.168.1.10

3. 公网客户端发出的请求到达网关路由器，在开始路由之前（PREROUTING），将IP转换为内网地址，公网客户端访问到内网服务器

 

**DNAT规则语法**

```bash
iptables -t nat -A PREROUTING -d <公网IP> -p tcp --dport <公网IP端口> -j DNAT --to-destination <内网主机IP:端口>
```

**DNAT规则案例**

```bash
iptables -t nat -A PREROUTING -d 192.168.2.107 -p tcp -m tcp --dport 22 -j DNAT --to-destination 192.168.3.2:22
```

```bash
iptables -t nat -A PREROUTING -s 1.2.3.4/32 -d 2.3.4.5/32 -j DNAT --to-destination 192.168.3.2
```



### SNAT规则

SNAT：源地址转换，在路由结束后，将数据包的源地址更换为外网段的IP地址



**POSTROUTING（路由后）详解：**

1. 前提：内网客户端192.168.1.10/24想通过地址转换到公网地址200.0.0.1/24

2. 此时内网客户端192.168.1.10想访问公网200.0.0.0/24网段

3. 内网客户端请求到达网关路由器，在完成路由后（POSTROUTING），将IP转换为公网地址，内网客户端访问到公网网段

 

**SNAT规则语法**

```bash
iptables -t nat -A POSTROUTING -s <内网IP或网段> -j SNAT --to-source <公网IP>    #适用于静态公网IP地址
```

```bash
iptables -t nat -A POSTROUTING -s <内网IP或网段> -j MASQUERADE    #适用于动态公网IP地址
```

```bash
iptables -t nat -A POSTROUTING -o <出口设备名> -j SNAT --to-source <出口设备IP地址>    #适用于所有网段，出口为指定设备的SNAT地址转换
```



**SNAT规则案例**

```bash
iptables -t nat -A POSTROUTING -s 192.168.3.0/24 -j SNAT --to-source 192.168.2.107
```

```bash
iptables -t nat -A POSTROUTING -s 192.168.3.0/24 -j MASQUERADE
```

```bash
iptables -t nat -A POSTROUTING -o em1 -j SNAT --to-source 192.168.2.1
```

```bash
iptables -t nat -A POSTROUTING -o em1 -j SNAT --to-source 192.168.2.1-192.168.2.110
```

 



## 实用案例

 

### 个人主机

```bash
service iptables stop    #关闭服务，清空当前配置
```

```bash
iptables -t filter -I INPUT -i lo -j ACCEPT
iptables -t filter -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -t filter -P INPUT DROP
iptables -t filter -P FORWARD DROP
#可根据自己的需要自行更改iptables规则
```

```bash
service iptables save    #保存当前防火墙配置
```

```bash
systemctl enable iptables    #设置开启自启
```



### 服务器

```bash
service iptables stop    #关闭服务，清空当前配置
```

```bash
iptables -t filter -I INPUT -i lo -j ACCEPT
iptables -t filter -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -t filter -A INPUT -p tcp -m multiport --dports 22,80,443 -j ACCEPT
iptables -t filter -A INPUT -p udp -m multiport --dports 53 -j ACCEPT
iptables -t filter -P INPUT DROP
iptables -t filter -P FORWARD DROP
#可根据自己的需要自行更改iptables规则
```
```bash
service iptables save    #保存当前防火墙配置
```

```bash
systemctl enable iptables    #设置开启自启
```

 







---



[^1]: 防火墙通过硬件或软件限制非法用户访问资源。防火墙工作在传输层（通过对不用软件标识所采用协议及端口、接收方采用相同协议及端口打开数据）

[^2]: nat 表中的INPUT链为CentOS7后新加入

