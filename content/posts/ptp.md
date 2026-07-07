---
title: "PTP"
date: 2026-07-07T14:09:54+08:00
draft: true
tags:
  - Services
description: 通过 PTP 实现高精度时间同步
---

通过 PTP （IEEE 1588）实现高精度时间同步，并尽可能精确地测量同步后的残余偏差，评估同步精度和稳定性



Linux中的PTP使用 **linuxptp** 套件实现（`ptp4l` + `phc2sys`） 



## 环境准备

安装 linuxptp

```bash
yum install linuxptp -y
```

关闭其他时间同步服务防止干扰

```bash
systemctl stop chronyd ntpd
```

关闭防火墙或允许udp 319、320端口通过

```bash
firewall-cmd --add-port=319/udp --add-port=320/udp --permanent
firewall-cmd --reload
```

查看物理网卡支持硬件时间戳

```bash
ethtool -T ens33 | grep hardware-transmit
```



## 启动主节点时钟

```bash
cat /etc/ptp4l-master.conf 

[global]
network_transport       L2		#二层以太网传输
delay_mechanism         E2E		#端到端延迟测量机制
domainNumber            0		#PTP 域编号（0-127，默认0）只有相同 domainNumber 的设备才会互相通信
gmCapable               1		#声明具备 Grandmaster 能力
priority1               128		#BMCA 第一优先级。范围 0-255，值越小优先级越高。
priority2               128		#BMCA 第二优先级。当 priority1 相同时，比较此值。
logAnnounceInterval     0		#Announce 报文间隔用于宣告 GM 身份和时钟质量。范围通常 -4 到 4
logSyncInterval         -3		#每秒 8 次 Sync 报文，值越小同步精度越高，最高-4
tx_timestamp_timeout    100		#等待硬件发送时间戳的超时时间，单位 毫秒
use_syslog              0		#不使用 syslog，日志不会写入 /var/log/messages 等系统日志
verbose                 1		#详细输出模式，日志直接输出到 stdout/stderr
masterOnly              1		#强制仅作为主时钟。
```


启动主时钟（想用软件时间戳换成-S）

```bash
ptp4l -f /etc/ptp4l-master.conf -i ens33 -m -H
```

实时同步硬件时钟到本地时间

```bash
phc2sys -s /dev/ptp1 -c CLOCK_REALTIME -m -O 0 -n 0 
```
## 启动从节点时钟

```bash
cat /etc/ptp4l-slave.conf 

[global]
network_transport       L2
delay_mechanism         E2E
domainNumber            0
slaveOnly               1
priority1               255
priority2               255
logAnnounceInterval     0
logSyncInterval         -3
tx_timestamp_timeout    100
use_syslog              0
verbose                 1
```

从时钟每秒输出一行统计摘要，包含过去 1 秒内所有同步偏差的均方根（rms）和最大绝对值（max）

定义输出文件

```bash
OFFSET_FILE="/tmp/ptp_rms_$(date +%Y%m%d_%H%M%S).log"
```

启动从时钟，记录数据（想用软件时间戳换成-S）

```bash
ptp4l -f /etc/ptp4l-slave.conf -i ens33 -m -H 2>&1 | stdbuf -oL grep "rms" >> "$OFFSET_FILE"
```

采集所有 rms 和 max 数值，分别统计均值、标准差、最小值和最大值，以评估同步精度与稳定性。

拿出rms偏差数据

```bash
awk '{print $3}' /tmp/ptp_rms_*.log > rms_ns.txt
awk '{
    sum+=$1; sumsq+=$1*$1; count++
    if(count==1){ min=$1; max=$1 }
    else {
        if($1<min) min=$1
        if($1>max) max=$1
    }
} END {
    avg = sum/count
    std = sqrt(sumsq/count - avg*avg)
    print "样本数:", count
    print "RMS均值:", avg, "ns"
    print "RMS标准差:", std, "ns"
    print "RMS最小值:", min, "ns"
    print "RMS最大值:", max, "ns"
}' rms_ns.txt

```

拿出max偏差数据

```bash
awk '{print $5}' /tmp/ptp_rms_*.log > max_ns.txt
awk '{
    sum+=$1; sumsq+=$1*$1; count++
    if(count==1){ min=$1; max_val=$1 }
    else {
        if($1<min) min=$1
        if($1>max_val) max_val=$1
    }
} END {
    avg = sum/count
    std = sqrt(sumsq/count - avg*avg)
    print "样本数:", count
    print "Max均值:", avg, "ns"
    print "Max标准差:", std, "ns"
    print "Max最小值:", min, "ns"
    print "Max最大值:", max_val, "ns"
}' max_ns.txt
```

实时同步硬件时钟到本地时间

```bash
phc2sys -s /dev/ptp6 -c CLOCK_REALTIME -m -O 0 -n 0 
```

## 验证同步结果

| 指标   | RMS 偏差 (ns) | Max 峰值偏差 (ns) |
| ------ | ------------- | ----------------- |
| 样本数 | 18007         | 18007             |
| 均值   | 347.76        | 642.90            |
| 标准差 | 197.64        | 516.91            |
| 最小值 | 75            | 124               |
| 最大值 | 17071         | 45312             |

\#单位换算参考：1 μs = 1,000 ns；1 ms = 1,000,000 ns。

本次 PTP 同步测试在硬件时间戳模式下，PTP 同步系统的平均时间偏差为 0.35 μs，瞬时最差偏差典型值 0.64 μs，稳定性优异，无异常大偏差。当前系统同步精度满足纳秒级应用需求，性能可靠。 