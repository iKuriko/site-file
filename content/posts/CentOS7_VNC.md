---
title: "CentOS7 VNC"
date: 2021-11-03T17:38:40+08:00
draft: true
tags:
  - CentOS Linux
description: 
---



安装"GNOME Desktop"软件包组

```bash
yum groupinstall -y "GNOME Desktop" 
```

 

安装完成后，修改默认启动方式为图形化界面

```bash
systemctl set-default graphical.target
```

 

如果要换回命令模式 

```bash
systemctl set-default multi-user.target 
```

 

然后重启系统即可

```bash
shutdown -r now
```

 

安装tigervnc

```bash
yum install tigervnc-server -y
```

 

过滤软件包是否安装

```bash
rpm -qa|grep tigervnc-server
```

 

拷贝服务启动脚本，其中1表示端口号，默认5900+，1即是5901，以此类推：

```bash
cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
```

 

编辑服务启动脚本，设置登录使用的用户，这里使用的是`root`用户

```bash
vim /etc/systemd/system/vncserver@\:1.service
```

 

```bash
 32 [Unit]
 33 Description=Remote desktop service (VNC)
 34 After=syslog.target network.target
 35 
 36 [Service]
 37 Type=forking
 38 
 39 # Clean any existing files in /tmp/.X11-unix environment
 40 ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'
 41 ExecStart=/usr/sbin/runuser -l root -c '/usr/bin/vncserver %i'
 42 PIDFile=/root/.vnc/%H%i.pid
 43 ExecStop=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'
 44 
 45 [Install]
 46 WantedBy=multi-user.target
```

 

设置vnc密码，（“View-only password”，只允许查看,无控制权限，这个没啥用，不用设置。）

```bash
vncpasswd
```

 

开启vnc服务

```bash
systemctl start vncserver@\:1.service
```

 

重新加载配置文件

```bash
systemctl daemon-reload
```

 

开启自启动

```bash
systemctl enable vncserver@\:1.service
```

 

过滤服务端口是否存在（无输出可再次重启服务尝试）

```bash
netstat -utpln | grep Xvnc
```

 

允许firewalld防火墙通过

```bash
firewall-cmd --zone=public --add-port=5901/tcp --permanent
```

 

重新加载生效

```bash
firewall-cmd --reload
```

 

查看与VNC相关的SELinux域策略有哪些

```bash
getsebool -a | grep vnc

xdm_bind_vnc_tcp_port --> off
```

 

修改SELinux策略，-P永久生效

```bash
setsebool -P xdm_bind_vnc_tcp_port=on
```

