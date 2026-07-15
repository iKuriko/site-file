---
title: "Rsyslog 日志集中收集配置"
date: 2026-05-19T10:06:57+08:00
draft: true
tags:
  - Services
description: 收集本地日志上传至中心服务器或作为中心日志服务器接收远程日志
---



## 服务端配置文件

 

server配置文件位置

```bash
/etc/rsyslog.d/10-server.conf
```

 

```bash
# ===== 模板：按主机名/程序名存储 =====
template(name="RemoteLog" type="string"
         string="/mnt/rsyslog/%HOSTNAME%/%PROGRAMNAME%.log")

# ===== 所有远程日志使用该模板 =====
if ($fromhost-ip != "127.0.0.1") then {
    action(type="omfile"
           dynaFile="RemoteLog"
           createDirs="on"
           dirCreateMode="0750"
           fileCreateMode="0640")
    stop
}

# ===== 接收远程日志（TCP 推荐） =====
module(load="imtcp"
       MaxSessions="2000"
       MaxListeners="20")

input(type="imtcp"
      port="514"
      address="0.0.0.0")


# 如需 UDP，可额外开启（不推荐）
# module(load="imudp")
# input(type="imudp" port="514")
```



 rsyslog 的 AppArmor 配置文件位置

```bash
/etc/apparmor.d/usr.sbin.rsyslogd
```



```bash
# Last Modified: Sun Sep 25 08:58:35 2011
#include <tunables/global>

# Debugging the syslogger can be difficult if it can't write to the file
# that the kernel is logging denials to. In these cases, you can do the
# following:
# watch -n 1 'dmesg | tail -5'

profile rsyslogd /usr/sbin/rsyslogd {
  #include <abstractions/base>
  #include <abstractions/nameservice>

  capability sys_tty_config,
  capability dac_override,
  capability dac_read_search,
  capability setuid,
  capability setgid,
  capability sys_nice,
  capability syslog,

  unix (receive) type=dgram,
  unix (receive) type=stream,

  # rsyslog configuration
  /etc/rsyslog.conf r,
  /etc/rsyslog.d/ r,
  /etc/rsyslog.d/** r,
  /{,var/}run/rsyslogd.pid{,.tmp} rwk,
  /mnt/rsyslog/** rw,
  /mnt/rsyslog/ rw,

  # LP: #2056768
  /{,var/}run/systemd/sessions/ r,
  /{,var/}run/systemd/sessions/* r,

  # LP: #2073628
  @{run}/log/journal/ r,
  /etc/machine-id r,

  /var/spool/rsyslog/ r,
  /var/spool/rsyslog/** rwk,

  /usr/sbin/rsyslogd mr,
  /usr/lib{,32,64}/{,@{multiarch}/}rsyslog/*.so mr,

  /dev/tty*                     rw,
  /dev/xconsole                 rw,
  @{PROC}/kmsg                  r,
  # allow access to console (LP: #2009230)
  /dev/console                  rw,

  /dev/log                      rwl,
  /{,var/}run/utmp              rk,
  /var/lib/*/dev/log            rwl,
  /var/spool/postfix/dev/log    rwl,
  /{,var/}run/systemd/notify    w,

  # 'r' is needed when using imfile
  /var/log/**                   rw,

  # LP: #2061726
  @{PROC}/sys/net/ipv6/conf/all/disable_ipv6 r,

  # apparmor snippets for rsyslog from other packages
  include if exists <rsyslog.d>

  # Site-specific additions and overrides. See local/README for details.
  #include <local/usr.sbin.rsyslogd>
}
```



将目录所有者改为 syslog:syslog（rsyslog服务的运行用户）

```bash
sudo chown -R syslog:syslog /mnt/rsyslog
```

修改目录权限，让 syslog 用户可以读写，组成员和其他用户只能读

```bash
sudo chmod -R 755 /mnt/rsyslog
```

验证修改结果

```bash
ls -la /mnt/rsyslog/
```

## 客户端配置文件



client配置文件位置

```bash
/etc/rsyslog.d/60-forward-to-master.conf 
```



```bash
# 客户端配置 - 转发日志到中央服务器

# 工作目录
$WorkDirectory /var/spool/rsyslog

# 将所有日志通过TCP转发到中央服务器
# @@ 表示TCP, @ 表示UDP
*.* @@服务端IP地址:514

# 可选：配置转发队列（防止服务器不可用时丢失日志）
$ActionQueueFileName forwardq      # 队列文件前缀
$ActionQueueMaxDiskSpace 1g        # 最大磁盘空间1GB
$ActionQueueSaveOnShutdown on      # 关机时保存队列
$ActionQueueType LinkedList        # 链表队列
$ActionResumeRetryCount -1         # 无限重试

```



## 启动服务

```bash
systemctl restart syslog
```

```bash
systemctl status syslog
```



#客户端发送测试消息 

```bash
logger -t "SYSLOG_TEST" "测试消息来自客户端 - $(date)" 
```

