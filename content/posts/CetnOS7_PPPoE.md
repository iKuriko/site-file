---
title: "CetnOS7 PPPoE Server"
date: 2022-12-30T10:29:28+08:00
draft: true
tags:
  - CentOS Linux
  - Linux services
  - PPPOE
description: 在CentOS7上搭建PPPoE Server
---

## 安装软件包

```
yum -y install rp-pppoe
```

## 修改配置文件

vim /etc/ppp/pppoe-server-options

```
# PPP options for the PPPoE server
# LIC: GPL
require-chap
require-pap
auth
login
lcp-echo-interval 10
lcp-echo-failure 2.4.4
logfile /var/log/pppoe-server.log
ms-dns 114.114.114.114
ms-dns 8.8.8.8
```

vim /etc/ppp/pppoe-server-env

```
INT=enp1s0
LOCAL=192.168.1.1
START=192.168.1.2
NUMBER=253
```

## 添加用户

vim /etc/ppp/chap-secrets

```
# Secrets for authentication using CHAP
# client        server  secret                  IP addresses
"test" * "123456" *
```

## 服务启动文件

vim /etc/systemd/system/pppoe-server.service

```
[Unit]
Description=PPPoE Server.
After=syslog.target

[Service]
Type=forking
EnvironmentFile=/etc/ppp/pppoe-server-env
ExecStart=/sbin/pppoe-server -I $INT -L $LOCAL -R $START -N $NUMBER

[Install]
WantedBy=multi-user.target
```

## 启动服务&开机自启

```
systemctl start pppoe-server
systemctl enable pppoe-server
```
