---
title: "CentOS8 PPPoE 4G"
date: 2022-03-09T11:44:50+08:00
draft: true
tags:
  - CentOS Linux
  - Linux services
  - PPPOE
description: 使用Huawei ME906s 4G模块PPPE拨号上网
---

1.插上4G模块，通过以下命令查看是否连接，输出如下

```bash
ls /dev/ttyUSB*
/dev/ttyUSB0 /dev/ttyUSB1 /dev/ttyUSB2 /dev/ttyUSB3 /dev/ttyUSB4
```

```bash
dmesg | grep ttyUSB
[  8.058470] usb 1-1.3: GSM modem (1-port) converter now attached to ttyUSB0
[  8.058700] usb 1-1.3: GSM modem (1-port) converter now attached to ttyUSB1
[  8.069734] usb 1-1.3: GSM modem (1-port) converter now attached to ttyUSB2
[  8.069918] usb 1-1.3: GSM modem (1-port) converter now attached to ttyUSB3
[  8.084378] usb 1-1.3: GSM modem (1-port) converter now attached to ttyUSB4
```

```bash
lsusb
Bus 001 Device 005: ID 12d1:15c1 Huawei Technologies Co., Ltd. ME906s LTE M.2 Module
Bus 001 Device 004: ID 05e3:0608 Genesys Logic, Inc. Hub
Bus 001 Device 002: ID 8087:07e6 Intel Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

2.新建拨号脚本文件夹

```bash
mkdir /etc/ppp/peers/
```

3.拨号的脚本

```bash
vim /etc/ppp/peers/gprsdial
```

```bash
# This is pppd script for China Mobile, used Huawei ME906s GPRS Module

/dev/ttyUSB0
9600
crtscts
modem
#noauth
debug
nodetach
#hide-password
usepeerdns
noipdefault
defaultroute
user "cmnet"
0.0.0.0:0.0.0.0
#ipcp-accept-local
#ipcp-accept-remote
#lcp-echo-failure 12
#lcp-echo-interval 3
#noccp
#novj
#novjccomp
#persist
connect '/usr/sbin/chat -s -v -f /etc/ppp/gprs-connect-chat'
#disconnect '/bin/chat -v -f /etc/ppp/gprs-disconnect-chat'
```

4.校验文本/etc/ppp/gprs-connect-chat的具体内容如下：

```bash
vim /etc/ppp/gprs-connect-chat
```

```bash
#Chat script for China Mobile, used Huawei ME906s LTE M.2 Module.

TIMEOUT 15
ABORT "DELAYED"
ABORT "BUSY"
ABORT "ERROR"
ABORT "NO DIALTONE"
ABORT "NO CARRIER"
TIMEOUT 40
'' \rAT
OK ATS0=0
OK ATE0V1
OK AT+CGDCONT=1,"IP","CMNET"
OK AT+CGEQREQ=1,2,128,384,,,0,,,,,,
OK ATDT*99*1#
CONNECT
```

5.拨号验证（两种方式效果相同）：

```bash
pppd call gprsdial &
或
pppd file /etc/ppp/peers/gprsdial &
```

6.编写拨号脚本，一键拨号，获取DNS

```bash
vim gprs_dialup.sh
```

```bash
#!/bin/sh
dns1=" "
dns2=" "
cd /etc/ppp/peers
pppd call gprsdial >/dev/null &
echo "pppd ok"
sleep 12
echo "sleep ok"
cp -rf /run/ppp/resolv.conf /etc/
sed -n '1p' /etc/resolv.conf > /etc/ppp/primarydns
sed -n '2p' /etc/resolv.conf > /etc/ppp/seconddns
dns1=`cut -f 2 -d ' ' /etc/ppp/primarydns`
dns2=`cut -f 2 -d ' ' /etc/ppp/seconddns`
echo $dns1
echo $dns2
echo "dns ok"
```

---

gprsdial参数详情

```bash
# This is pppd script for China Mobile, used Huawei GTM900-B GPRS Module
/dev/ttyUSB0  #端口名称Modem port for ppp-dial
9600  #通信波特率
crtscts  #接口带硬件流控  nocrtscts(无流控制)
modem  #使用数据机控制线路。这个选项是默认的。硬体流控，pppd将等待CD信号。
noauth  #不需要对方验证自己
debug  #把调试信息输出到/var/log/messages
nodetach  #不后台运行，默认是后台运行的
hide-password  #写log内容时不包括密码字符串，这个参数是默认的
usepeerdns  #选中这个选项，从对方请求两个DNS地址. 对方提供的地址传给文件/etc/ppp/ip-up中的环境变量DNS1和DNS2,将环境变量USEPEERDNS设置成1. 而且pppd将创建一个文件/etc/ppp/resolv.conf file，其中一个或两个服务器行包括由对方提供的地址。
noipdefault  #关闭在没有指定本地IP位址时所进行的预设动作，这是用来由从主机名称决定（如果可能的话）本地IP位址。加上这个选项的话，彼端将必须在进行IPCP协商时（除非在指令行或在选项档中明确地指定它）提供本地的IP地址。
defaultroute      #当 IPCP 协商完全成功时， 增加一个预设递送路径到系统的递送表，将彼端当作闸道器使用。这个项目在 ppp 连线中断後会移除。
refuse-chap       #外置模块刷新DNS
refuse-mschap
refuse-mschap-v2
user "cmnet"      #用户，中国移动。设置由对方验证本地系统的用户名。
0.0.0.0:0.0.0.0
ipcp-accept-local      #加上这个选项的话，pppd将会接受彼端对於本地IP位址的意见，即使本地的IP位址已经在某个选项中指定。
ipcp-accept-remote        #加上这个选项的话，pppd将会接受彼端对於它的IP位址的意见，即使远端的IP位址已经在某个选项中指定。
lcp-echo-failure 12    #如果有给这个选项，那麽如果传送n个LCP回应要求没有接收到有效的LCP回应回覆的话pppd将会推测彼端是死掉的。如果发生这种情形，pppd将会终结该连线。这个选项的使用要求一个非零的lcp-echo-interval参数值。这个选项可以用在硬体数据机控制线路无法使用的情况下当实际连线被中断之後（e.g.,数据机已经挂断）终结 pppd的执行。
lcp-echo-interval 3    #如果有给这个选项，pppd每秒将会送出一个LCP回应要求(echo-request)封包(frame)给彼端。在Linux系统下，回应要求在n秒内没有从彼端接收到封包时会被送出。一般彼端应该以传送一个回应回覆(echo-reply)来反应该回应要求。这个选项可以与lcp-echo-failure选项一起使用来侦测不再连线的彼端。
noccp        #关闭压缩控制协议协商。若对方有漏洞会被来自PPPD的压缩控制协议协商请求干扰的情况下，需要设置该选项。
novj        #选中这个选项，将关闭双方的Van Jacobson形式TCP/IP报文头压缩
novjccomp    #选中这个选项，将关闭Van Jacobson形式TCP/IP报文头压缩中的连接ID压缩。Pppd将忽略来自VanJacobson形式压缩TCP/IP报文头中的连接ID字节，也不要求对方这样作。
persist    #连接中断后不退出，而是重新打开连接。
connect '/usr/sbin/chat -s -v -f /etc/ppp/gprs-connect-chat'  #拨号调用一个验证脚本，账号加密用的
disconnect '/bin/chat -v -f /etc/ppp/gprs-disconnect-chat'
```

---

References & Resources：

[linux环境下pppd gprs拨号上网总结](https://blog.csdn.net/heiniaoyuyouling/article/details/5694792)

[在ARM-linux上实现4G模块PPP拨号上网](https://blog.csdn.net/zqixiao_09/article/details/52540887)

[etc/ppp/gprs文件详解](http://blog.chinaunix.net/uid-31410005-id-5783842.html)
