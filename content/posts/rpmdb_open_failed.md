---
title: "Rpmdb open failed"
date: 2026-02-12T16:42:54+08:00
draft: true
tags:
  - Other
description: rpm 被玩坏时，报错 ‘Error: Error: rpmdb open failed’ 问题解决
---



rpm报错现象如下

```bash
…………

error: rpmdb: BDB0113 Thread/process 65218/139733911608192 failed: BDB1507 Thread died in Berkeley DB library

error: db5 error(-30973) from dbenv->failchk: BDB0087 DB_RUNRECOVERY: Fatal error, run database recovery

error: cannot open Packages index using db5 -  (-30973)

error: cannot open Packages database in /var/lib/rpm

Error: Error: rpmdb open failed
```



删除 rpm 数据库

```bash
cd /var/lib/rpm
```

```bash
rm -f __db.*  
```



新建 rpm 数据库

```bash
rpm --rebuilddb
```

 

刷新仓库元数据 

```bash
dnf update --refresh
```



删除所有yum缓存

```bash
rm -r /var/cache/dnf
```

