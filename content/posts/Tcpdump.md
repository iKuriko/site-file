---
title: "Tcpdump"
date: 2021-10-11T17:30:36+08:00
draft: true
tags:
  - CentOS Linux
  - Network tools
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
| -n       | 不会解析主机名                                               |
| -nn      | 不会解析主机名或端口，例如ssh直接输出为:22                   |
| -e       | 显示连接层级的协议报头                                       |
| -w       | 导出为文件，后缀一般为 .cap                                  |
| -c       | 可以指定抓取数据包的数量                                     |
| -l       | 指定行缓冲模式，可以让输出立即发送到管道命令                 |
| -C       | 指定数据包缓冲模式，可以让输出立即发送到管道命令             |
| -s       | 抓包大小。 -s 0会将大小设置为无限制                          |
| -v       | 详细使用 (-v) 或 (-vv) 会增加输出中显示更详细信息，通常会显示更多协议特定的信息 |
| -A       | 输出中包含抓包的 `ascii` 字符串                              |
| -X       | 同时显示十六进制输出和 `ascii` 字符串                        |
| host     | 指定抓取的主机，包括 src host 和 dst host                    |
| src host | 源主机                                                       |
| dst host | 目标主机                                                     |
| and      | 用来衔接多个选项                                             |
| &&       | 与and用法一致                                                |
| or       | 用来并列多个选项                                             |
| \|\|     | 与or用法一致                                                 |
| tcp      | TCP协议                                                      |
| udp      | UDP协议                                                      |
| port     | 指定抓取的端口号                                             |
| not      | 不抓取指定的主机/协议/端口，如 not port 22 ：不抓取22号端口的数据包 |
| !        | 与not用法一致                                                |
| proto    | 指定抓取的协议号，如：抓取UDP流量时，写作 proto 17           |



## 常用命令示例

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

  

导出为文件

```bash
tcpdump -c 100 -i ens33 -w ens33.cap
```



查看导出的文件中的tcp协议22端口的流量

```bash
tcpdump -nr ens33.cap tcp port 22
```



提取 HTTP 请求的 URL（如果服务不在80 端口，则需要指定端口）

```bash
tcpdump -s 0 -v -n -l | egrep -i "POST /|GET /|Host:"
```
