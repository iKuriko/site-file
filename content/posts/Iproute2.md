---
title: "Iproute2"
date: 2022-04-02T14:23:55+08:00
draft: true
tags:
  - Tools
description: Iproute2 是 Linux 下强大的网络配置工具，用来显示或配置 Linux 主机的路由、网络设备、策略路由和隧道等功能。
---





## 简介

Iproute2 是Linux 下管理控制TCP/IP网络和流量控制的新一代工具包，旨在代替老派的工具包 net-tools（ifconfig,arp,route,netstat等命令）

这两套工具本质的区别是，net-tools 是通过 procfs（/proc）和 ioctl 系统调用去访问和改变内核中的网络配置，而 iproute2 则通过 netlink套接字接口与内核通讯


Iproute2 用来显示或操纵Linux主机的路由，网络设备，策略路由和隧道，是Linux下强大的网络配置工具  

　

net-tools 和 iproute2 常用命令对比

| 功能               | 老用法           | 新用法           |
| ------------------ | ---------------- | ---------------- |
| 网络连接           | netstat -a       | ss               |
| 路由表             | netstat -r/route | ip route         |
| 网络接口统计信息   | netstat -I       | ip -s link       |
| 组播成员           | netstat -g       | ip maddr         |
| 伪连接             | netstat -M       | ss               |
| 网络接口地址和链路 | Ifconfig         | ip addr /ip link |
| ARP                | arp              | ip neigh         |
| 隧道               | Iptunnel         | ip tunnel        |




## 命令安装

```bash
yum install -y iproute iproute-doc    #安装工具包和文档
```

```bash
ip --help
Usage: ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip [ -force ] -batch filename
where  OBJECT := { link | address | addrlabel | route | rule | neigh | ntable |
                   tunnel | tuntap | maddress | mroute | mrule | monitor | xfrm |
                   netns | l2tp | fou | macsec | tcp_metrics | token | netconf | ila |
                   vrf }
       OPTIONS := { -V[ersion] | -s[tatistics] | -d[etails] | -r[esolve] |
                    -h[uman-readable] | -iec |
                    -f[amily] { inet | inet6 | ipx | dnet | mpls | bridge | link } |
                    -4 | -6 | -I | -D | -B | -0 |
                    -l[oops] { maximum-addr-flush-attempts } | -br[ief] |
                    -o[neline] | -t[imestamp] | -ts[hort] | -b[atch] [filename] |
                    -rc[vbuf] [size] | -n[etns] name | -a[ll] | -c[olor]}
```

　


基本用法：

```bash
ip address show  | ip a    //显示分配给所有网络接口的地址
```

```bash
ip link show  |  ip l   //显示所有网络设备的运行状态
```

```bash
ip neigh show  |  ip n   //显示当前的邻居表
```

```bash
ip route show  |  ip r    //显示核心路由表
```

```bash
ip rule show  |  ip ru    //显示当前路由规则
```

```bash
ip address [add|del] 192.168.0.1/24 dev ens33    //为接口添加|删除IP地址
```

```bash
ip link set ens33 [up|down]    //启动|关闭接口
```

```bash
ip -s link show ens33    //显示接口统计
```

```bash
ss -utpln    //显示当前监听的进程
```

```bash
ip ro add blackhole 1.2.3.4/32    //黑洞路由，来自这个IP的数据包不做应答
```





## iproute策略路由

iproute2 策略路由比传统路由在功能上更强大，使用更灵活。它能够根据目的地址或源地址，报文大小、应用等需求来选择数据包的转发路径。

　

iproute2 策略路由分为两个部分：**路由转发表**和**路由匹配规则**。通过编写路由匹配规则，选择相应的路由表，进行数据的查表转发

1. 路由转发表：ip route

2. 路由匹配规则：ip rule


　

策略路由是把数据信息从源穿过网络到达目的地的行为，有两个动作：

1. 确定最佳路径：手动指定路由或自动学习路由
2. 传输信息：隧道传输，流量整形

　



### 添加路由表

**多路由表**

策略路由一般手工添加路由表，在rt_tables文件中存放了表名和表号的对应关系，以数字来区分路由表。iproute2 默认存在编号为255、254、253三张表，表0为保留，iproute2 最多支持255张表

```bash
cat /etc/iproute2/rt_tables

# reserved values

255	local    // 本地路由表，存有本地接口地址，广播地址，以及NAT地址
254	main    // 主路由表，传统路由表,ip route若没有指定路由表，则默认操作该表（一般存在所有主机的路由条目）
253	default    // 默认路由表一般存放默认路由
0	unspec    // 系统保留

# local
#1	inr.ruhep
```

添加表名和表号的对应关系

```bash
vim /etc/iproute2/rt_tables

+ 252	table1
```

添加路由条目

```bash
ip route add 192.168.2.0/24 via 192.168.1.1 table table1    //添加静态路由到表1，优先匹配最小掩码
ip route add default via 192.168.1.1 table table1    //添加默认路由到表1，默认的路由是转发的最后保障，匹配不到路由时，会匹配默认路由
```

```bash
ip route flush cache    //刷新路由缓冲，立即生效
```

查看路由表中的路由条目

```bash
ip route show table table1
default via 192.168.88.1 dev ens33
192.168.2.0/24 via 192.168.88.1 dev ens33 
```

\* 路由表中应当指明默认路由，尽量不回查路由表，路由添加完毕，即可在路由规则中应用



### 添加转发规则



路由在选路时，数据包根据路由规则进行匹配。按照优先级（pref）从高到低匹配（数字越小，优先级越大），直到找到合适的规则。如果没有匹配到合适的路由条目，则会匹配默认路由，所以在应用中配置默认路由是必要的。

 　

添加转发规则 

```bash
ip rule add pref 1000 from 192.168.1.0/24 lookup table1    //从192.168.1.0/24发来的包，按照编号为1的表进行匹配路由
ip rule add pref 1000 to 192.168.2.0/24 lookup table1    //去192.168.2.0/24的包，按照编号为1的表进行匹配路由
ip rule add pref 800 from 192.168.1.0/24 to 192.168.1.0/24 lookup table1    //从192.168.1.0/24来的，要去192.168.1.0/24的包，按照编号为1的表进行匹配路由
```

查看转发规则

```bash
ip rule show
0:	from all lookup local
800:	from 192.168.1.0/24 to 192.168.1.0/24 lookup table1
1000:	from 192.168.1.0/24 lookup table1
1000:	from all to 192.168.2.0/24 lookup table1
32766:	from all lookup main
32767:	from all lookup default
```



ip rule 参数详解：

```bash
from -- 源地址 
to -- 目的地址（这里是选择规则时使用，查找路由表时也使用） 
tos -- IP包头的TOS（type of sevice）域 
dev -- 物理接口 
fwmark -- 防火墙参数 
采取的动作除了指定路由表外，还可以指定下面的动作：
nat -- 透明网关 
prohibit -- 丢弃该包，并发送 COMM.ADM.PROHIITED的ICMP信息 
reject -- 单纯丢弃该包 
unreachable -- 丢弃该包，并发送 NET UNREACHABLE的ICMP信息 
```



**数据包标记转发**

```bash
iptables -t mangle -A PREROUTING -p tcp -m multiport --dports 80,8080,20,21 -s 192.168.1.0/24 -j MARK --set-mark 1    //将目的端口为tcp,80,8080,20,21，源IP为192.168.1.0/24的数据包标记为1
```

```bash
ip rule add fwmark 1 pref 1000 table table1    //标记为1的数据包进行查表转发
```



## iptunnel隧道



GRE是通用路由封装协议，内部能封装二层，三层，MPLS……它能封装的报文类型很多，但一般都用于封装三层报文，所以也有叫三层隧道协议的，但这个说法也不严谨。例如：GRE能封装MPLS，ISIS等，这些都不是三层报文。  



### 三层GRE

ip 命令创建（一次性，重启失效）

```bash
ip tunnel add gre01 mode gre remote 192.168.2.22 local 192.168.2.23 ttl 255    //创建ip隧道，类型为三层gre，名称为gre01，远程IP地址2.215，本地IP地址2.180
```

```bash
ip addr add 192.168.0.1/30 dev gre01    //给gre01配置IP地址
```

```bash
ip link set gre01 up    //启动gre01接口
```

配置文件配置

```bash
vim /etc/sysconfig/network-scripts/ifcfg-gre01
```

```bash
DEVICE=gre01
BOOTPROTO=none
ONBOOT=yes
TYPE=GRE
PEER_OUTER_IPADDR=192.168.2.22
MY_OUTER_IPADDR=192.168.2.23
PEER_INNER_IPADDR=192.168.0.2/30
MY_INNER_IPADDR=192.168.0.1/30
TTL=255
```



### 二层GRE

```bash
ip link add gretap01 mode gretap remote 192.168.2.215 local 192.168.2.180 ttl 255    //创建ip隧道，类型为二层gretap，名称为gretap01，远程IP地址2.22，本地IP地址2.23
```

```bash
ip addr add 192.168.0.1/30 dev gretap01    //给gretap01配置IP地址
```

```bash
ip link set dev gretap01 up    //启动gretap01接口
```

　

ps：

如果源目地址相同，想要创建多条gre隧道，需要添加key值的选项

```bash
ip link add gretap01 mode gretap remote 192.168.2.215 local 192.168.2.180 ttl 255 key 1
```

删除隧道

```bash
ip link del <隧道名称>
```

