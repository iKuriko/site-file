---
title: "SSH No Password"
date: 2022-04-15T17:27:16+08:00
draft: true
tags:
  - Other
description: 生成 ssh 密钥对，实现客户端到服务器的免密 ssh 用户登录
---



Linux的远程连接使用ssh协议：安装openssh软件包，监听22端口；默认安装并启动sshd服务





## SSH认证方式

（1）密码认证：用户名、密码登陆

（2）密钥对登陆：客户端生成密钥对，将公钥上传至服务器，实现免密登录至服务器



## 配置SSH免密登录



全局配置文件

```bash
vim /etc/ssh/sshd_config
48 PubkeyAuthentication yes			//启用公钥认证
49 AuthorizedKeysFile      .ssh/authorized_keys	//指定公钥文件路径
```



两种方式实现（效果相同）

1.使用ssh-copy-id（ssh-copy-id只能通过22端口拷贝到目标主机）

```bash
ssh-keygen -t rsa -b 1024			//生成rsa 1024位算法的密钥对
```

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub <user>@<IP>	//将本地公钥文件上传到指定服务器的指定用户下
```



2.使用scp（可以使用自定义的端口）

Client端

```bash
scp -P 20333 ~/.ssh/id_rsa.pub <user>@<IP>:~/.ssh/    //将本地的公钥文件远程拷贝到指定的服务器目录下
```

Server端

```bash
chown -R <user>:<group> ~/.ssh    //设置目录的所属者、组，如果没有该目录，需提前新建：mkdir ~/.ssh
```

```bash
chmod 700 ~/.ssh    //设置目录权限
```

```bash
touch ~/.ssh/authorized_keys    //新建认证密钥文件
```

```bash
chmod 600 ~/.ssh/authorized_keys    //设置文件权限
```

```bash
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys    //导入公钥文件，等待客户端登录，多个客户端连接，可以多次追加导入
```



## 登录测试

Client端

```bash
ssh <user>@<Server IP>
```

　

Ps：

1、客户端如需多个用户连接服务器，在每个用户下新建.ssh目录，并将authorized_keys拷贝到.ssh目录，设置所有者、组  
2、如需多个客户端连接服务器，使用：~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys，这样实现客户端公钥文件追加
