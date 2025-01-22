---
title: "WireGuard"
date: 2021-12-31T10:06:44+08:00
draft: true
tags:
  - VPN
  - Linux services
description: WireGuard 是一个简单、快速且安全的开源VPN软件 
---

## 简介

WireGuard 利用最新的加密技术，提供更快、更精简、更安全、易于部署、方便管理的通用VPN，可以在低端到高端设备上部署。

仅 4000 行代码的精简代码库。将它与 OpenVPN（另一个流行的开源 VPN）的 100,000 行代码相比。显然调试 WireGuard 更加容易。

支持所有最新的加密技术，例如 Noise 协议框架、Curve25519、ChaCha20、Poly1305、BLAKE2、SipHash24、HKDF 和安全受信任结构。

运行在内核空间，因此可以高速的提供安全的网络。

这些是 WireGuard 越来越受欢迎的一些原因。Linux 创造者 Linus Torvalds 非常喜欢 WireGuard，以至于将其合并到 Linux Kernel 5.6 中以取代OpenVPN。

## 安装

添加软件包仓库源

```bash
yum install epel-release elrepo-release yum-plugin-elrepo -y
```

安装WireGuard及相关依赖包

```bash
yum install kmod-wireguard wireguard-tools qrencode -y
```

## IPv4模式

### 服务端配置（Server）

**创建wireguard接口**

生成server端公钥（publickey）和私钥（privatekey）（类似SSH）

```bash
wg genkey | tee ./test_server.pri | wg pubkey > ./test_server.pub
```

创建wg0接口

```bash
ip link add wg0 type wireguard
```

启动接口设置监听的端口和私钥

```bash
wg set wg0 listen-port 42333 private-key ./test_server.pri
```

配置IP地址

```bash
ip addr add 198.18.233.1/24 dev wg0
```

**添加客户端**

> WireGuard没有明确的服务端和客户端的定义，他们是对等的一对peer，这里为了方便理解，模拟了一台服务器和一台PC机的账号创建。将服务器命名为client1，终端设备PC命名为client2。

为client1（另一台服务端）生成公钥和私钥

```bash
wg genkey | tee ./test_client1.pri | wg pubkey > ./test_client1.pub
```

为client2（终端设备）生成客户端的公钥和私钥

```bash
wg genkey | tee ./test_client2.pri | wg pubkey > ./test_client2.pub
```

查看客户端的公钥和私钥，之后配置需使用

```bash
cat ./test_client[1/2].[pub/pri]
```

在Server端添加client1的peer公钥和允许的IP

```bash
wg set wg0 peer $(cat ./test_client1.pub) allowed-ips 198.18.233.2/32
```

在Server端添加client2的peer公钥和允许的IP

```bash
wg set wg0 peer $(cat ./test_client2.pub) allowed-ips 198.18.233.3/32
```

删除客户端

```bash
wg set wg_qlzy peer  $(cat ./test_client.pub) remove
```

**保存配置**

新建wg0配置文件

```bash
touch /etc/wireguard/wg0.conf
```

保存wg0配置

```bash
wg-quick save wg0
```

**启用接口**

启动接口（有配置文件的接口才能使用）

```bash
wg-quick up wg0
```

关闭接口（同上）

```bash
wg-quick down wg0
```

开机自启动wg0

```bash
systemctl enable wg-quick@wg0.service
```

### 客户端配置（Client1）

创建wireguard接口

```bash
ip link add wg0 type wireguard
```

配置client1的私钥、server的公钥、允许的IP、服务端的IP和端口号

```bash
wg set wg0 private-key ./test_client1.pri peer $(test_server.pub) persistent-keepalive 1 allowed-ips 0.0.0.0/0 endpoint server_ip:server_port
```

配置IP地址

```bash
ip addr add 198.18.233.2/24 dev wg0
```

启动wg0接口

```bash
ip link set wg0 up
```

添加默认路由

```bash
ip ro add default via 198.18.233.1
```

SNAT策略（如有必要）

```bash
iptables -t nat -A POSTROUTING -o wg0 -j SNAT --to-source 198.18.233.2
```

### 终端设备配置（Client2）

client2的配置文件

```bash
#test_client.conf
[Interface]
PrivateKey = 0KyR/4U48Xzyfhqxa76UNHgrGg0TSLaATctaVwFSQxQe2ls=    #test_client2.pri
Address = 198.18.233.3/32
DNS = 8.8.8.8
MTU = 1420
[Peer]
PublicKey = 2Cg4wDfeiYnc2dhxW99y7nlmKROR5zfxB2hFtjBmi4aoMjFo=    #test_server.pub
Endpoint = 213.55.77.88:54233
AllowedIPs = 198.18.233.0/24
PersistentKeepalive = 1
```

#可以用配置文件生成二维码 

```bash
qrencode -o test_client.png < test_client.conf
```

## IPv6模式

> Wireguard使用IPv6的配置与IPv4并无太大差异，这里只做简单演示

### 服务端配置（Server）

**创建wireguard接口**

生成server端公钥（publickey）和私钥（privatekey）（类似SSH）

```bash
wg genkey | tee ./test_server.pri | wg pubkey > ./test_server.pub
```

创建wg1接口

```bash
ip link add wg1 type wireguard
```

启动接口设置监听的端口和私钥

```bash
wg set wg1 listen-port 42333 private-key ./test_server.pri
```

配置IP地址

```bash
ifconfig wg1 inet6 add fec0:1001:1001:1001:1001:1001:1234:0001/112
```

启用接口

```bash
ip link set wg1 up
```

**添加客户端**

为client生成客户端的公钥和私钥

```bash
wg genkey | tee ./test_client3.pri | wg pubkey > ./test_client3.pub
```

查看客户端的公钥和私钥，之后配置需使用

```bash
cat ./test_client3.[pub/pri]
```

在Server端添加client3的peer公钥和允许的IP

```bash
wg set wg1 peer allowed-ips fec0:1001:1001:1001:1001:1001:1234:0002/128
```

### 客户端配置（Client3）

创建wg1接口

```bash
ip link add wg1 type wireguard
```

配置client3的私钥、server的公钥、允许的IP、服务端的IP和端口号

```bash
wg set wg1 private-key $(test_client3.pri) peer $(test_server.pub) persistent-keepalive 1 allowed-ips 0:0:0:0:0:0:0:0/0 endpoint server_ip:server_port
```

配置IP地址

```bash
ifconfig wg1 inet6 add fec0:1001:1001:1001:1001:1001:1234:0002/112
```

启用wg1接口

```bash
ip link set wg1 up
```

添加默认路由

```bash
ip -6 ro add default via fec0:1001:1001:1001:1001:1001:1234:0001
```

SNAT策略（如有必要）

```bash
ip6tables -t nat -A POSTROUTING -o wg1 -j SNAT --to-source fec0:1001:1001:1001:1001:1001:1234:0002
```
