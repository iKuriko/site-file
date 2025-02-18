---
title: "L2TP/IPsecVPN"
date: 2025-02-18T09:36:22+08:00
draft: true
tags:
  - VPN
  - Linux services
description: Linux 下的 L2TP（Layer 2 Tunneling Protocol）通常与 IPsec 结合使用以提供加密和安全性。
---



## 安装依赖包

```bash
yum install -y epel-release
yum install -y libreswan xl2tpd ppp
```


## 配置IPSec（Libreswan）

**1. 修改`/etc/ipsec.conf`**

```bash
vim /etc/ipsec.conf
```

```bash
version 2.0
config setup
    virtual_private=%v4:10.0.0.0/8,%v4:192.168.0.0/16,%v4:172.16.0.0/12
    protostack=netkey
    interfaces=%defaultroute

conn L2TP-PSK
    authby=secret
    pfs=no
    auto=add
    keyingtries=3
    rekey=no
    ikelifetime=8h
    keylife=1h
    type=transport
    left=%defaultroute
    leftprotoport=17/1701
    right=%any
    rightprotoport=17/%any
    dpddelay=30
    dpdtimeout=120
    dpdaction=clear
```

**2. 设置预共享密钥**

```bash
openssl rand -base64 16  # 生成一个16字节的随机字符串，可以把16改为32，加强密钥安全性
```

```bash
vim /etc/ipsec.secrets
```

```bash
%any  %any  : PSK "YOUR_KEY（刚才生成的预共享密钥）"
```

```bash
chmod 600 /etc/ipsec.secrets
```



## 配置L2TP（xl2tpd）

**1. 修改`/etc/xl2tpd/xl2tpd.conf`**

```bash
vim /etc/xl2tpd/xl2tpd.conf
```

```bash
[global]
ipsec saref = yes
listen-addr = <服务器IP>    #替换为服务器接入IP地址
#port = 1701

[lns default]
ip range = 100.88.88.2-100.88.88.254
local ip = 100.88.88.1
require chap = yes
refuse pap = yes
require authentication = yes
ppp debug = yes
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
EOF
```

**2. 配置PPP选项**

```bash
vim /etc/ppp/options.xl2tpd
```

```bash
require-mschap-v2
ms-dns 8.8.8.8    # 为客户端分发DNS
ms-dns 8.8.4.4
asyncmap 0
auth
crtscts
lock
hide-password
modem
debug
name l2tpd
proxyarp
lcp-echo-interval 30
lcp-echo-failure 4
mtu 1400
mru 1400
nobsdcomp
defaultroute    # 为客户端分发默认网关
```


## 配置用户认证

```bash
vim /etc/ppp/chap-secrets
```

```bash
# 格式：用户名 服务类型 密码 分配IP
vpnuser1  *  "Passw0rd@123"  *
```

```bash
chmod 600 /etc/ppp/chap-secrets
```



## 启动服务

```bash
systemctl enable --now ipsec xl2tpd
```

>  注意：实际部署时请替换所有`<服务器IP>`和`YOUR_KEY`为真实值，建议使用更复杂的密码组合。生产环境建议启用firewalld并仅开放必要端口（UDP 500/4500/1701）。



## 客户端连接参数

- 服务器IP：服务器IP
- 预共享密钥：YOUR_KEY
- 用户名/密码：vpnuser1/Passw0rd@123
- 客户端将获得：
  - IP地址：100.88.88.2-254
  - 默认网关：100.88.88.1（通过VPN隧道）
  - DNS：8.8.8.8 和 8.8.4.4




## 测试连接
**1. Windows 10/11 配置步骤**

**打开VPN设置**

- 点击任务栏右下角的网络图标（Wi-Fi/以太网）。
- 选择 **"网络和Internet设置"**。
- 在左侧菜单中选择 **"VPN"**。
- 点击 **"添加VPN连接"**。

**填写VPN连接信息**

- **VPN提供商**：选择 **"Windows (内置)"**。
- **连接名称**：任意命名（例如：MyL2TPVPN）。
- **服务器名称或地址**：填写您的L2TP服务器公网IP地址。
- **VPN类型**：选择 **"使用预共享密钥的L2TP/IPSec"**。
- **预共享密钥**：填写您在服务器上设置的预共享密钥（`YOUR_KEY`）。
- **登录信息的类型**：选择 **"用户名和密码"**。
- **用户名**：填写您在服务器上设置的用户名（例如：`vpnuser1`）。
- **密码**：填写您在服务器上设置的密码（例如：`Passw0rd@123`）。
- 勾选 **"记住我的登录信息"**（可选）。
- 点击 **"保存"**。

**修改VPN连接属性**

- 在VPN连接列表中，找到刚刚创建的VPN连接。
- 点击 **"高级选项"**。
- 在 **"VPN属性"** 中，确保以下设置：
  - **类型**：L2TP/IPSec with 预共享密钥。
  - **数据加密**：可选（默认即可）。
  - **身份验证**：CHAP 和 MS-CHAP v2。
- 点击 **"确定"** 保存。

**连接VPN**

- 返回VPN连接列表。
- 点击刚刚创建的VPN连接，然后点击 **"连接"**。
- 如果一切配置正确，VPN连接将成功建立。



------

**2. Windows 7/8 配置步骤**

**打开网络和共享中心**

- 点击任务栏右下角的网络图标。
- 选择 **"打开网络和共享中心"**。

**创建新的VPN连接**

- 点击 **"设置新的连接或网络"**。
- 选择 **"连接到工作区"**，然后点击 **"下一步"**。
- 选择 **"使用我的Internet连接 (VPN)"**。

**填写VPN连接信息**

- **Internet地址**：填写您的L2TP服务器公网IP地址。
- **目标名称**：任意命名（例如：MyL2TPVPN）。
- 勾选 **"现在不连接，仅进行设置以便稍后连接"**（可选）。
- 点击 **"下一步"**。

**填写登录信息**

- **用户名**：填写您在服务器上设置的用户名（例如：`vpnuser1`）。
- **密码**：填写您在服务器上设置的密码（例如：`Passw0rd@123`）。
- 勾选 **"记住此密码"**（可选）。
- 点击 **"创建"**。

**修改VPN连接属性**

- 返回 **"网络和共享中心"**。
- 点击左侧的 **"更改适配器设置"**。
- 找到刚刚创建的VPN连接，右键点击并选择 **"属性"**。
- 在 **"安全"** 选项卡中：
  - **VPN类型**：选择 **"使用IPSec的第2层隧道协议 (L2TP/IPSec)"**。
  - **数据加密**：可选（默认即可）。
  - **身份验证**：勾选 **"Microsoft CHAP 版本 2 (MS-CHAP v2)"**。
- 点击 **"高级设置"**：
  - 选择 **"使用预共享密钥进行身份验证"**。
  - 填写您在服务器上设置的预共享密钥（`YOUR_KEY`）。
  - 点击 **"确定"**。
- 点击 **"确定"** 保存。

**连接VPN**

- 返回 **"网络和共享中心"**。
- 点击左侧的 **"连接到网络"**。
- 选择刚刚创建的VPN连接，点击 **"连接"**。
- 如果一切配置正确，VPN连接将成功建立。



**3. 连接成功后，打开命令提示符（`cmd`）。**


```bash
ipconfig    # 确认是否获取到服务器分配的IP地址（例如：`100.88.88.x`）。
```

```bash
ping 100.88.88.1  # 服务器隧道IP
ping 8.8.8.8      # 测试DNS
```





## 常见问题排查

**1. 检查服务状态：**

```bash
ipsec verify
```

**问题1：**
Disable /proc/sys/net/ipv4/conf/*/send_redirects or NETKEY will act on or cause sending of bogus ICMP redirects!

```bash
#解决方法：
for iface in /proc/sys/net/ipv4/conf/*/send_redirects; do
    echo 0 > $iface
done
```

**问题2：**
Disable /proc/sys/net/ipv4/conf/*/accept_redirects or NETKEY will act on or cause sending of bogus ICMP redirects!

```bash
#解决方法：
for iface in /proc/sys/net/ipv4/conf/*/accept_redirects; do
    echo 0 > $iface
done
```

```bash
systemctl status ipsec xl2tpd
```

**2. 查看日志：**

```bash
tail -f /var/log/messages
```

```bash
journalctl -u ipsec
```

**3. 抓包排查：**

```bash
tcpdump -i eth0 udp port 500 or port 4500
```

**4. 注意事项：**

- 确保服务器的IPSec服务已启动。
- 检查预共享密钥是否正确。
- 确保防火墙关闭或UDP 500和4500端口在服务器防火墙中已开放。
- 检查连接使用的用户名和密码是否正确。
- 确保服务器上的`/etc/ppp/chap-secrets`文件配置正确。
- 确保服务器已启用IP转发并配置了正确的SNAT规则。
- 检查客户端是否获取了正确的默认网关和DNS。

