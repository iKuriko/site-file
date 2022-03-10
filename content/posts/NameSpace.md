---
title: "Network NameSpace"
date: 2022-02-22T14:50:30+08:00
draft: true
tags:
  - Linux
  - Namespace
description:
---

## NameSpace 简介


Network NameSpace：网络命名空间（名字空间），是 Linux 内核中隔离资源的一种特性，是实现 Linux 网络虚拟化的重要功能，它能创建多个隔离的网络空间，它们拥有独自的网络栈信息，包括网络设备接口、ipv4,v6 协议栈、路由表、防火墙规则、sockets 等。不管是虚拟机还是容器，运行的时候仿佛自己就在独立的网络中。

## NameSpace 使用

创建一个命名空间

```bash
ip netns add netns1
```

创建的命名空间会在以下目录生成文件

```bash
ls /var/run/netns/
netns1
```

对命名空间使用命令

```bash
ip netns exec <ns-name> <cmd>
```

**Example：**

通过bash进入network namespace 并更新命令行提示符

```bash
ip netns exec netns1 bash
```

网络命名空间包含自己的网络资源：接口，路由表等。（默认添加的网络命名空间netns1中会自动添加一个loopback(回环)接口但需要手动开启）

```bash
ip netns exec netns1 ip link set lo up
```

```bash
ip netns exec netns1 ifconfig lo 127.0.0.1/32 up
```

```bash
ip netns exec netns1 ping 127.0.0.1 -c 3
```

查看网络接口信息

```bash
ip netns exec netns1 ip addr
```

添加路由

```bash
ip netns exec netns1 ip route add 192.168.2.0/24 via 192.168.1.1 dev eth0
```

添加接口

```bash
ip link set veth0 netns netns1
```

启动接口

```bash
ip netns exec netns1 ip link set veth0 up 
```



## NameSpace 连接方式

在 Linux 内核中与 NameSpace 关联性较强的技术是 LXC 。LXC[^1]（Linux Container）是一种内核虚拟化技术，它可以提供轻量级的虚拟化服务，以便隔离系统进程和系统资源。Linux Container 技术使用 cgroups（control groups）和 namespace 实现。两者的功能如下： 

- Cgroups（资源限制）

- Namespace（资源隔离）

  

在使用KVM（Kernel-based Virtual Machine）或LXC（Linux Container）时，典型的物理主机不会为KVM或每个容器中的虚拟机的每一个网口（NIC，network interface controller）提供一个或多个物理适配器，必须通过互连虚拟网络接口实现通信。这里使用的经典工具是 Linux 网桥（Linux bridge），它内置在 Linux 的内核中，非常古老，管理 Linux bridge 的前端管理工具是 brctl，能实现的功能类似二层交换机，功能非常有限。而较新的模拟工具是 OpenvSwitch，主要的前端管理是 ovs-vsctl，它能实现更多更复杂的功能。

因为使用 ip tuntap | ip link add 创建的 linux tap 接口不能用于将网络命名空间链接到 Linux bridges 或 openvswitch ，所以提供了以下的方法进行 namespace 的连接：



### veth pair

连接两个网络命名空间的简单解决方案是使用一对 veth 接口，veth pair(一对虚拟网口)

![ns1.jpg](/images/ns/ns1.png)



使用 veth 连接 namespaces

新建命名空间

```bash
ip netns add ns1
ip netns add ns2
```

创建 veth pair

```bash
ip link add veth0 type veth peer name veth1
```

连接命名空间的端口,加入 veth-pair

```bash
ip link set veth0 netns ns1
ip link set veth1 netns ns2
```

启动连接

```bash
ip netns exec ns1 ip link set dev veth0 up
ip netns exec ns2 ip link set dev veth1 up
```





### Linux bridge 和 2个 veth pair



当必须连接两个以上的网络命名空间（Netns或KVM或LXC实例）时，应使用虚拟交换机。Linux 提供了众所周知的Linux 网桥作为一种解决方案

![ns2.jpg](/images/ns/ns2.png)



使用 Linux 网桥和两个 veth pair连接网络命名空间

对于此种设置，我们需要一个交换机和两条连接线（虚拟意义上的）。使用一个 Linux bridge 和两个 veth pair

添加命名空间

```bash
ip netns add ns1
ip netns add ns2
```

创建Linux 网桥

```bash
brctl addbr $BRIDGE
brctl stp $BRIDGE off
ip link set dev $BRIDGE up
```

端口 port 1

创建一个 veth 端口对

```bash
ip link add veth0 type veth peer name br-veth0
```

一边加入 Linux 网桥

```bash
brctl addif br-test br-veth0
```

另一边加入命名空间

```bash
ip link set veth0 netns ns1
```

开启连接

```bash
ip netns exec ns1 ip link set dev veth0 up
ip link set dev br-veth0 up
```

端口 port 2

创建一个 veth 端口对

```bash
ip link add veth1 type veth peer name br-veth1
```

一边加入 Linux 网桥

```bash
brctl addif br-test br-veth1
```

另一边加入命名空间

```bash
ip link set veth1 netns ns2
```

开启连接

```bash
ip netns exec ns2 ip link set dev veth1 up
ip link set dev br-verh1 up
```







### OpenvSwitch 和 2 个 veth pair

另一种解决方案是使用 openvswitch 而不是旧的 Linux bridge。配置与 Linux bridge 相同

![ns3.jpg](/images/ns/ns3.png)

使用 ovs 和 2 个 veth pair 连接 namespaces

对于此设置，需要一个交换机和两条连接线（虚拟意义上的）。使用一个 ovs 网桥和两个 veth pair

添加命名空间

```bash
ip netns add ns1
ip netns add ns2
```

创建ovs网桥

```bash
ovs-vsctl add-br $BRIDGE
```

端口 port 1

创建一个veth端口对

```bash
ip link add veth0 type veth peer name ovs-veth0
```

一边端口加入 ovs 网桥

```bash
ovs-vsctl add-port $BRIDGE ovs-veth0
```

另一端端口加入命名空间

```bash
ip link set veth0 netns ns1
```

开启连接

```bash
ip netns exec ns1 ip link set dev veth0 up
ip link set dev ovs-veth0 up
```

端口 port 2

创建一个 veth 端口对

```bash
ip link add veth1 type veth peer name ovs-veth1
```

一边端口加入 ovs 网桥

```bash
ovs-vsctl add-port $BRIDGE ovs-veth1
```

另一端端口加入命名空间

```bash
ip link set veth1 netns ns2
```

开启连接

```bash
ip netns exec ns2 ip link set dev veth1 up
ip link set dev ovs-veth1 up
```



### OpenvSwitch 和 2 个 OpenvSwitch 端口

另一种解决方案是使用 openvswitch 并利用 openvswitch 内部端口。这避免了必须在所有其他解决方案中使用的veth pair 的使用

![ns4.jpg](/images/ns/ns4.png)

使用 openvswtich 和 2 个 openvswitch  端口连接 namespaces

对于此设置，需要一个交换机和两条连接线（虚拟意义上的）。使用一个 openvswitch 和两个 openvswitch 端口

添加命名空间

```bash
ip netns add ns1
ip netns add ns2
```

创建 ovs 网桥

```bash
ovs-vsctl add-br $BRIDGE
```

端口 port 1
创建 ovs 内部端口

```bash
ovs-vsctl add-port $BRIDGE tap1 --set interface tap1 type=internal
```

把它加到命名空间

```bash
ip link set tap1 netns ns1
```

启动连接

```bash
ip netns exec ns1 ip link set dev tap1 up
```

端口 port 2
创建 ovs 内部端口

```bash
ovs-vsctl add-port $BRIDGE tap2 --set interface tap2 type=internal 
```

把它加到命名空间

```bash
ip link set tap2 netns ns2
```

启动连接

```bash
ip netns exec ns2 ip link set dev tap2 up
```



---

[^1]: Linux container技术的目标是为应用程序或系统提供完整的资源隔离和控制。LXC项目通过提供一组API接口和工具，可以让其他程序方便地使用Linux container技术。











