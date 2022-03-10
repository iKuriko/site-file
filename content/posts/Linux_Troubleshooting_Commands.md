---
title: "Linux 常用排查命令"
date: 2021-11-16T11:01:07+08:00
draft: true
tags:
  - Linux
description:
---



## ps

ps命令：可以查看进程的瞬间信息。

```bash
ps -aux
```

常用参数：

-a：显示现行终端机下的所有程序，包括其他用户的程序。

-e：列出程序时，显示每个程序所使用的环境变量。

-f：显示UID,PPIP,C与STIME栏位。



查看非root用户运行的进程

```bash
ps -U root -u root -N
```

查看户root运行的进程

```bash
ps -u root
```

查看有没有奇怪进程

```bash
ps -aef | grep inetd        #inetd 程序是一个Linux守护进程.
```

检测隐藏进程(ps)

```bash
ps -ef | awk '{print}' | sort -n |uniq >1
```

检查系统硬件及当前正在运行进程的信息[^1]

```bash
ls /proc | sort -n | uniq > 2
```





## pstree

查看进程树是否所有异常进程存在一个父进程、判断进程的父子关系

```bash
pstree -p
```



## last

查看近期用户登陆情况

```bash
last -n 5                  # -n 5 表示输出5条
```

## history

查看历史命令

```bash
history 5                  # 5 表示输出最近使用的5条命令
```



## awk

查看空口令账号awk[^2]

```bash
awk -F: '($2=="")' /etc/shadow
```



查看uid为0的账号

```bash
awk -F: '($3==0)' /etc/passwd
```



查看uid为0的账号

```bash
grep -v -E "^#" /etc/passwd | awk -F: '$3==0{print $1}'
#cat /etc/passwd | awk -F: '$3==0'
```

```bash
-v    --反转匹配选择不匹配的线
-E    --扩展正则表达式模式是一个扩展正则表达式
```

## netstat

查看服务使用的端口号

netstat 命令用来打印Linux中网络系统的状态信息。

```bash
netstat -utpln
```

常用参数：

-a或–all：显示所有连线中的Socket。

-c或–continuous：持续列出网络状态。

-i或–interfaces：显示网络界面信息表单。

-l或–listening：显示监控中的服务器的Socket。

-n或–numeric：直接使用ip地址，而不通过域名服务器。

-t或–tcp：显示TCP传输协议的连线状况。

-u或–udp：显示UDP传输协议的连线状况。



## lsof

查看谁在使用某个端口

lsof 命令用于查看你进程开打的文件，打开文件的进程，进程打开的端口(TCP、UDP)。

```bash
lsof
```

常用参数：

-g：列出GID号进程详情；

-d<文件号>：列出占用该文件号的进程；

-i<条件>：列出符合条件的进程。（4、6、协议、:端口、 @ip ）

-p<进程号>：列出指定进程号所打开的文件；

-u：列出UID号进程详情；



查看端口的使用信息

```bash
lsof -i :22       # 看看谁在使用22端口
```

查看多个进程号对应的文件信息

```bash
lsof -p 2,3   # 使用逗号分隔 
```

查看所有tcp网络连接信息

```bash
lsof -i tcp
```



## ls

ls 命令用来显示目标列表，不同类型的文件颜色也不同

```bash
ls
```

常用参数：

-a：显示所有文件及目录，包括隐藏文件

-l：以长格式显示目录下的内容列表。

-t：用文件和目录的更改时间排序

查看所有文件，包括隐藏的文件(ls)

```bash
ls -la
```



## whereis

查看文件的路径

```bash
whereis hostname

hostname: /usr/bin/hostname /etc/hostname ……
```



## find

find 命令用来在指定目录下查找文件。

```bash
find
```

参数 -mtime n 按照文件的更改时间来找文件，n为整数。

例：

-mtime 0 表示文件修改时间距离当前为0天的文件，即距离当前时间不到1天（24小时）以内的文件。

-mtime 1 表示文件修改时间距离当前为1天的文件，即距离当前时间1天（24小时－48小时）的文件。

-mtime＋1 表示文件修改时间为大于1天的文件，即距离当前时间2天（48小时）之外的文件

-mtime -1 表示文件修改时间为小于1天的文件，即距离当前时间1天（24小时）之内的文件

 

查找最近24小时内修改过的文件(find)

```bash
find ./ -mtime 0
```

查找以.txt结尾的文件名(find)

```bash
find / -name "*.txt"
```

忽略大小写：

```bash
find / -iname "*.txt"
```

查找不是以.txt结尾的文件(find)

```bash
find / ! -name "*.txt"
```



[^1]:sort 命令将文本文件内容加以排序,可针对文本文件的内容，以行为单位来排序。-n 参数依照数值的大小排序。uniq 命令用于检查及删除文本文件中重复出现的行列，一般与 sort 命令结合使用。
[^2]: awk语法： awk [options] ‘pattern{action}’ file    #awk是一种编程语言，用于对文本和数据进行处理的

 

 
