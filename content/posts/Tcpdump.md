---
title: "Tcpdump"
date: 2021-10-11T17:30:36+08:00
draft: true
tags:
  - CentOS Linux
  - Tcpdump
description: Linux 下软件包分析工具 Tcpdump 的常用命令
---





## 命令语法

```bash
tcpdump [选项] <抓取的网口> [选项]
```



| 选项     | 功能                                                         |
| -------- | ------------------------------------------------------------ |
| -D       | 列出所有可抓取的网口                                         |
| -i       | 指定抓取的网口，any表示所有网口                              |
| -n       | 不把主机的网络地址转换成名字。                               |
| -nn      | 以端口号显示协议，例如 ssh == 22                             |
| -c       | 指定抓取数据包的数量                                         |
| host     | 指定抓取的主机，包括 src host 和 dst host                    |
| src host | 源主机                                                       |
| dst host | 目标主机                                                     |
| and      | 用来衔接多个选项                                             |
| tcp      | TCP协议                                                      |
| udp      | UDP协议                                                      |
| port     | 指定抓取的端口号                                             |
| not      | 不抓取指定的主机/协议/端口，如 not port 22 ：不抓取22号端口的数据包 |



## 命令示例

列出所有可抓取的网口

```bash
tcpdump -D
```

  

抓取ens33网口，过滤所有数据包

```bash
tcpdump -nn -i ens33    
```

  

抓取ens33网口，过滤 ping 包

```bash
tcpdump -nn -i ens33 icmp
```



抓取所有网口，过滤源目主机为192.168.1.1的ping包

```bash
tcpdump -nn -i any host 192.168.1.1 and icmp
```



抓取所有网口，过滤源主机为192.168.1.1，目的主机位192.168.1.2的ping包

```bash
tcpdump -nn -i any src 192.168.1.1 and dst 192.168.1.2 and icmp
```



抓取所有网口，过滤源主机为192.168.1.1 & 端口号不是tcp22的数据包

```bash
tcpdump -nn -i any src host 192.168.1.1 and not tcp port 22
```

  

抓取所有网口，过滤目的主机为192.168.1.2 & 端口号不是udp53的数据包

```bash
tcpdump -nn -i any dst host 192.168.1.2 and not udp port 53
```

  

抓取100个ens33网卡的数据包

```bash
tcpdump -c 100 -i ens33
```

  

