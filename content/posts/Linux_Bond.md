---
title: "Linux Link Aggregation"
date: 2023-10-31T11:06:02+08:00
tags:
  - Network tools
  - CentOS Linux
draft: true
description: Linux通过bond&Team，可以很容易实现网口冗余，负载均衡，实现高可用高可靠的目的
---






## 链路聚合实现方式

RHEL6 之前的方式使用 bond 方式，最多支持添加两块网卡

RHEL7.3 之后使用 Team 方式，最多支持添加八块网卡。建议使用Team，但也同时兼容bond模式



## Bond 模式

bond mode共有七种bond0、bond1……bond6



**常用的有三种**

mode=0：轮询策略，有自动备援，需要交换机支持并配置链路聚合。

mode=1：主备策略，其中的一条线若断线，其他线路将会自动备援。

mode=6：适配器传输负载均衡，有自动备援，无需交换机支持和配置。





## Bond 配置案例(rhel 6)

物理网口：eth0、eth1

虚拟网口：bond0

**配置bond**

```bash
vim /etc/sysconfig/network-scripts/ifcfg-bond0

DEVICE=bond0
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.168.1.101
NETMASK=255.255.255.0
NETWORK=192.168.1.0
BROADCAST=192.168.1.255    //BROADCAST广播地址
USERCTL=no      //是否允许非root用户控制
BONDING_OPTS="mode=1 miimon=100"		//mode：bond的模式,miimon：监测相应的时间，这里是100ms
```

**配置eth0、eth1**

```bash
vim /etc/sysconfig/network-scripts/ifcfg-eth0

DEVICE=eth0
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
USERCTL=no
```

```bash
vim /etc/sysconfig/network-scripts/ifcfg-eth1

DEVICE=eth1
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
USERCTL=no
```

**修改modprobe相关设定文件，并加载bonding模块**(rhel 6.5)

```bash
vim /etc/modprobe.d/bonding.conf

alias bond0 bonding
#options bond0 max_bonds=2 miimon=200 mode=1	//如果在网卡配置文件中添加过bond选项，可忽略

#alias bond1 bonding
#options bond1 max_bonds=2 miimon=200 mode=1	//如果在网卡配置文件中添加过bond选项，可忽略

#选项：
#miimon：监视网络链接的频度，单位是毫秒，我们设置的是200毫秒。
#max_bonds：配置的bond口个数
#mode：bond模式，主要有以下几种，在一般的实际应用中，0和1用的比较多，
```

**加载bonding模块**

```bash
modprobe bonding		//加载模块
```

```bash
lsmod | grep bonding	//查看是否加载成功
```

```bash
echo "modprobe bonding" >> /etc/rc.local    //加入开机自启 
```

**重启网络服务**(rhel6.5)

```bash
/etc/init.d/network restart
```



**查看bond状态**

```bash
cat /proc/net/bonding/bond0

Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup)	//**active-backup(主备)模式**
Primary Slave: None
Currently Active Slave: eth1	//**active状态的网口是eth1**
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth0
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 1
Permanent HW addr: 00:0c:29:8a:f3:88
Slave queue ID: 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 00:0c:29:8a:f3:92
Slave queue ID: 0
```

**任意关掉一个网口，看看网络是否仍然连通**

```bash
ifdown eth0
```

**处于active状态下时eth0、eth1、bond1的物理地址都会相同，这样是为了避免上位交换机发生混乱。**

```bash
ifconfig | grep HWaddr

bond0     Link encap:Ethernet  HWaddr 00:0C:29:8A:F3:88  
eth0      Link encap:Ethernet  HWaddr 00:0C:29:8A:F3:88  
eth1      Link encap:Ethernet  HWaddr 00:0C:29:8A:F3:88 
```





## 七种bond模式说明

**第一种模式：mod=0 ，即：(balance-rr) Round-robin policy（平衡轮循环策略）**

链路负载均衡，增加带宽，支持容错，一条链路故障会自动切换正常链路，交换机需要配置聚合口（port channel）。

特点：传输数据包顺序是依次传输（即：第1个包走eth0，下一个包就走eth1….一直循环下去，直到最后一个传输完毕），此模式提供负载平衡和容错能力；但是我们知道如果一个连接或者会话的数据包从不同的接口发出的话，中途再经过不同的链路，在客户端很有可能会出现数据包无序到达的问题，而无序到达的数据包需要重新要求被发送，这样网络的吞吐量就会下降

**第二种模式：mod=1，即： (active-backup) Active-backup policy（主-备策略）**

主备模式，只有一块网卡是active，另一块是备用的standby，所有流量都在active链路上处理，交换机配置的是捆绑的话将不能工作，因为交换机往两块网卡发包，有一半包是丢弃的。

特点：只有一个设备处于活动状态，当一个宕掉另一个马上由备份转换为主设备。mac地址是外部可见得，从外面看来，bond的MAC地址是唯一的，以避免switch(交换机)发生混乱。此模式只提供了容错能力；由此可见此算法的优点是可以提供高网络连接的可用性，但是它的资源利用率较低，只有一个接口处于工作状态，在有 N 个网络接口的情况下，资源利用率为1/N

**第三种模式：mod=2，即：(balance-xor) XOR policy（平衡XOR策略）**

表示XOR Hash负载分担，和交换机的聚合强制不协商方式配合。（需要xmit_hash_policy，需要交换机配置port channel）

特点：基于指定的传输HASH策略传输数据包。缺省的策略是：(源MAC地址 XOR 目标MAC地址) % slave数量。其他的传输策略可以通过xmit_hash_policy选项指定，此模式提供负载平衡和容错能力

**第四种模式：mod=3，即：broadcast（广播策略）**

表示所有包从所有网络接口发出，这个不均衡，只有冗余机制，但过于浪费资源。此模式适用于金融行业，因为他们需要高可靠性的网络，不允许出现任何问题。需要和交换机的聚合强制不协商方式配合。

特点：在每个slave接口上传输每个数据包，此模式提供了容错能力

**第五种模式：mod=4，即：(802.3ad) IEEE 802.3ad Dynamic link aggregation（IEEE 802.3ad 动态链接聚合）**

表示支持802.3ad协议，和交换机的聚合LACP方式配合（需要xmit_hash_policy）标准要求所有设备在聚合操作时，要在同样的速率和双工模式，而且，和除了balance-rr模式外的其它bonding负载均衡模式一样，任何连接都不能使用多于一个接口的带宽。

特点：创建一个聚合组，它们共享同样的速率和双工设定。根据802.3ad规范将多个slave工作在同一个激活的聚合体下。

外出流量的slave选举是基于传输hash策略，该策略可以通过xmit_hash_policy选项从缺省的XOR策略改变到其他策略。需要注意的 是，并不是所有的传输策略都是802.3ad适应的，尤其考虑到在802.3ad标准43.2.4章节提及的包乱序问题。不同的实现可能会有不同的适应 性。

必要条件：

条件1：ethtool支持获取每个slave的速率和双工设定

条件2：switch(交换机)支持IEEE 802.3ad Dynamic link aggregation

条件3：大多数switch(交换机)需要经过特定配置才能支持802.3ad模式

**第六种模式：mod=5，即：(balance-tlb) Adaptive transmit load balancing（适配器传输负载均衡）**

根据每个slave的负载情况选择slave进行发送，接收时使用当前轮到的slave。该模式要求slave接口的网络设备驱动有某种ethtool支持；而且ARP监控不可用。

特点：不需要任何特别的switch(交换机)支持的通道bonding。在每个slave上根据当前的负载（根据速度计算）分配外出流量。如果正在接受数据的slave出故障了，另一个slave接管失败的slave的MAC地址。

该模式的必要条件：ethtool支持获取每个slave的速率

**第七种模式：mod=6，即：(balance-alb) Adaptive load balancing（适配器适应性负载均衡）**

在mod5的tlb基础上增加了rlb(接收负载均衡receive load balance)，不需要任何switch(交换机)的支持。接收负载均衡是通过ARP协商实现的.

特点：该模式包含了balance-tlb模式，同时加上针对IPV4流量的接收负载均衡(receive load balance, rlb)，而且不需要任何switch(交换机)的支持。接收负载均衡是通过ARP协商实现的。bonding驱动截获本机发送的ARP应答，并把源硬件地址改写为bond中某个slave的唯一硬件地址，从而使得不同的对端使用不同的硬件地址进行通信。

来自服务器端的接收流量也会被均衡。当本机发送ARP请求时，bonding驱动把对端的IP信息从ARP包中复制并保存下来。当ARP应答从对端到达 时，bonding驱动把它的硬件地址提取出来，并发起一个ARP应答给bond中的某个slave。使用ARP协商进行负载均衡的一个问题是：每次广播 ARP请求时都会使用bond的硬件地址，因此对端学习到这个硬件地址后，接收流量将会全部流向当前的slave。这个问题可以通过给所有的对端发送更新 （ARP应答）来解决，应答中包含他们独一无二的硬件地址，从而导致流量重新分布。当新的slave加入到bond中时，或者某个未激活的slave重新 激活时，接收流量也要重新分布。接收的负载被顺序地分布（round robin）在bond中最高速的slave上

当某个链路被重新接上，或者一个新的slave加入到bond中，接收流量在所有当前激活的slave中全部重新分配，通过使用指定的MAC地址给每个 client发起ARP应答。下面介绍的updelay参数必须被设置为某个大于等于switch(交换机)转发延时的值，从而保证发往对端的ARP应答 不会被switch(交换机)阻截。

必要条件：

条件1：ethtool支持获取每个slave的速率；

条件2：底层驱动支持设置某个设备的硬件地址，从而使得总是有个slave(curr_active_slave)使用bond的硬件地址，同时保证每个bond 中的slave都有一个唯一的硬件地址。如果curr_active_slave出故障，它的硬件地址将会被新选出来的 curr_active_slave接管



mod=6与mod=0的区别：mod=6，先把eth0流量占满，再占eth1，….ethX；而mod=0的话，会发现2个口的流量都很稳定，基本一样的带宽。而mod=6，会发现第一个口流量很高，第2个口只占了小部分流量









## Team 模式

| 模式         | 作用                                                         |
| ------------ | ------------------------------------------------------------ |
| broadcast    | 广播容错，设备通过所有端口传输数据包                         |
| roundrobin   | 负载轮询，以轮循的模式传输所有端口的包                       |
| activebackup | 主备模式，这是一个故障迁移程序，监控链接更改并选择活动的端口进行传输 |
| loadbalance  | 负载均衡，监控流量并使用哈希函数以尝试在选择传输端口的时候达到完美均衡 |
| lacp         | 链路聚合控制协议，需要交换机支持                             |



## Team 配置案例(rhel 7)

Team 需要 NetworkManager 支持

```bash
yum -y install NetworkManager
```

```bash
systemctl start NetworkManager && systemctl enable NetworkManager
```

可以通过使用`nmtui`命令进入图形化界面进行配置，也可以使用命令行`nmcli`进行配置，配置逻辑基本相同

创建组接口

```bash
nmcli con add type team con-name team0 ifname team0 config '{"runner":{"name":"roundrobin"}}'
```

```bash
nmcli con add type team-slave con-name team0-port1 ifname ens32 master team0
```

```bash
nmcli con add type team-slave con-name team0-port2 ifname ens33 master team0
```


JSON语法格式如下（使用`nmtui`进行配置时需要手动输入）

```bash
‘{“runner”：{“name”：“METHOD”}}’
```

为组接口配置IP地址、网关、DNS等参数

```bash
nmcli con mod team0 ipv4.addresses "192.168.100.100/24"
```

```bash
nmcli con mod team0 ipv4.gateway "192.168.100.1"
```

```bash
nmcli con mod team0 ipv4.dns "192.168.100.1"
```

```bash
nmcli con mod team0 ipv4.method manual
```

```bash
nmcli con up team0-port1
```

```bash
nmcli con up team0-port2
```

```bash
nmcli con up team0
```

查看组接口状态

```bash
teamdctl team0 state
```

列出team0中的端口

```bash
teamnl team0 ports
```





