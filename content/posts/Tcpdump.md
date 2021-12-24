---
title: "tcpdump 常用命令"
date: 2021-10-11T17:30:36+08:00
draft: true
---







<p style="text-align:left;color:red;font-size:20px;font-family:arial">Linux下抓包工具Tcpdump的常用方法</p>  



列出所有可抓取的网口

```bash
tcpdump -D
```

  

以端口号显示协议，抓取ens33网卡的数据包

```bash
tcpdump -nn -i ens33
```

  

抓取100个ens33网卡的数据包

```bash
tcpdump -c 100 -i ens33
```

  

抓取10个所有网卡过滤源主机为192.168.1.1，端口号不是tcp22的数据包

```bash
tcpdump -nn -c 10  src host  192.168.1.1 and not tcp port 22 -i any
```

  

抓取10个所有网卡过滤目标主机为192.168.1.2，端口号不是udp22的数据包

```bash
tcpdump -nn -c 10 dst host 192.168.1.2 and not udp port 22 -i any
```

  

抓取ping包

```bash
tcpdump -nn -i ens33 -c 10 icmp
```



