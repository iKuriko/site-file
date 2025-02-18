---
title: "Acme.sh aliyun"
date: 2024-12-16T17:27:49+08:00
draft: true
tags:
  - Tools
description: 使用 acme.sh 为阿里云域名申请免费的 ssl 证书
---



## 安装 acme.sh

```bash
curl https://get.acme.sh | sh
```



## 配置阿里云 API 密钥

1、登录阿里云控制台

2、进入 Access Key 管理创建一个新的 Access Key

3、记录 Access Key ID 和 Access Key Secret

官方教程：[如何在阿里云创建AccessKey](https://help.aliyun.com/zh/ram/user-guide/create-an-accesskey-pair)



## 申请证书

```bash
cd /root/.acme.sh

#输入你刚刚申请的阿里云密钥
export Ali_Key="你的Access Key ID"
export Ali_Secret="你的 Access Key Secret"

#设置邮箱
./acme.sh --register-account -m 你的邮箱

#申请证书
./acme.sh --issue --dns dns_ali -d 你的域名  -k 4096
```



## 续签

默认证书有效期为三个月，会在到期前一个月自动续签

```bash
cd /root/.acme.sh
#查看下次续签的时间
./acme.sh --renew -d 你的域名 
```

你也可以手动强制续签

```bash
./acme.sh --renew -d 你的域名 --force
```



## 常见问题

当你遇到“*you* *don't* *specify* *aliyun* *api* *key* *and* *secret* *yet*.”这个提示时，通常意味着你在使用阿里云的服务或*API*时，没有正确配置*API* *Key*和*Secret*。



