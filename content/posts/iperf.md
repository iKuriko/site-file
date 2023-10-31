---
title: "Iperf"
date: 2022-03-25T11:38:58+08:00
draft: true
tags:
  - Network tools
  - CentOS Linux
description: iperf 是一个网络性能测试工具。可以测试最大TCP和UDP的带宽性能，具有多种参数，可以根据需要进行调整，可以报告带宽、延迟抖动和数据包丢失。
---

## 命令安装

**CentOS 安装 iperf2&3**

```bash
yum -y install epel-release
```

```bash
yum -y install iperf iperf3
```

**iperf2 & iperf3**

- iperf 存在两个版本，iperf2与iperf3。iperf2 的软件包 在 CentOS8 中被官方软件仓库移除。

- iperf3不支持双工模式测试，iperf2支持双工测试[^1]

- iperf3 server端使用了统一的命令`iperf3 -s`，不再区分测试的是UDP还是TCP

- 相同硬件条件下（x86平台），iperf3能测试的UDP带宽更高，但丢包严重。iperf2能测试的UDP带宽较小，但不丢包

## 命令语法

```bash
iperf3 [-c/-s] <服务端IP地址> <其他选项>
```

| 选项  | 作用                     |
| --- | ---------------------- |
| -t  | 设置时间，默认10s             |
| -u  | 使用UDP协议，默认为TCP         |
| -c  | 指定服务端的IP地址             |
| -b  | 设置带宽大小。带宽越大，每秒钟发送的数据越多 |
| -P  | 设置多线程的数量               |
| -p  | 指定端口号                  |
| -s  | 启用服务端                  |
| -l  | 指定数据包的长度[^2]           |
| -B  | 绑定使用的C\|S端的IP地址        |
| -f  | 显示的流量单位 [B/K/M/G]      |
| -R  | 反向传输测试                 |
| -d  | 双工传输测试                 |
| -w  | 设置tcp窗口大小              |
| -i  | 报告的时间间隔，单位是秒           |

## 命令示例

两台设备，其中一端开启服务端准备接收数据包，另一端启用客户端进行测试

**Server端**

```bash
iperf3 -s -B 192.168.88.140 -p 52001
```

命令详解：

-s 指定服务端 

-p 指定端口（要和客户端一致）为 25001

-B 绑定IP地址为 192.168.33.103

#-u udp协议，默认是tcp协议，iperf3的server端不支持“-u”参数，默认可以测试tcp和udp

**Client端**

```bash
iperf3 -c 192.168.88.140 -b 100M -t 10s -P 4 -u -p 52001
```

命令详解：

-c 指定服务端的IP地址

-b 指定流量大小为 100M

-t 设置时间为10s

-P 使用 4 个线程同时发送

-u 使用UDP协议

-p 指定服务器的端口为 25001

---

[^1]: iperf 2.05 客户端可以使用参数"-d"来进行双工测试，先测试发送，client向server发送数据，等到测试时间结束后（默认为10s，可以通过-t选项来更改），然后再测试接收，client端接收server发送数据，最后得出发送和接收吞吐率。

[^2]: <1> TCP默认为8KB，UDP默认为1470字节。测试时，需要保证被测试网卡的MTU值 > 测试包的长度，即 -l 的值，默认UDP的packet size是1470，加上udp和ip头的长度28， 等于1498。若默认的packet size > MTU，将会出现接收端收不到数据的情况。<2> 增加包的长度或增大缓冲区长度可以减少丢包率，因为包长度很小的话会造成包的数量更多，更易造成拥塞。在发送包为大包情况下，保障不丢包的方式，应当同时增大系统的读写缓冲区大小（修改/etc/sysctl.conf）

References & Resources：

[iperf2 与 iperf3 的区别，及使用与介绍](https://blog.csdn.net/hfut_zhanghu/article/details/122980059)
