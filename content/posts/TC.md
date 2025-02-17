---
title: "TC"
date: 2025-01-17T16:58:12+08:00
draft: true
tags:
  - Tools
description: 使用 wondershaper 工具对 Linux 下的网络接口限速
---







## 网络接口限速

```bash
#安装依赖软件包（基于tc命令实现）
yum install iproute-tc						
```

```bash
#下载 wondershaper 软件包
git clone https://github.com/magnific0/wondershaper.git     
```

```bash
#添加快捷软链接
ln -s /root/wondershaper-master/wondershaper /usr/local/sbin/wondershaper	
```

添加限速策略

```bash
#-a（接口）、-d(下载) 、-u（上传） 单位（kb）
wondershaper -a 接口名 -d 下载带宽 -u 上传带宽			
```

查看限速策略

```bash
wondershaper -s -a 接口名
```

删除限速策略

```bash
wondershaper -c -a 接口名
```



使用示例

```bash
#对enp1s0接口的下载带宽进行限速，限速为20MB
wondershaper -a enp1s0 -d 20480 -u 20480    
```



## 模拟时延和丢包率



```bash
#安装依赖软件包
yum install iproute-tc kernel-modules-extra -y    
```

```bash
#模拟网卡延迟（100ms为设定值，20ms为浮动值）
tc qdisc add dev enp2s0 root netem delay 100ms 20ms
```

```bash
#模拟网卡丢包率（25%）
tc qdisc add dev enp2s0 root netem loss  25%
```

```bash
# 查看网卡传输配置
tc qdisc show dev enp2s0
```



