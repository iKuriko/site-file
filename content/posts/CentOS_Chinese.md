---
title: "CentOS Chinese"
date: 2021-11-05T10:34:28+08:00
draft: true
tags:
  - CentOS Linux
  - Other
description: 为 CentOS 系统安装中文语言支持
---

安装中文语言包

```bash
yum install kde-l10n-Chinese          # CentOS 7 安装
```

```bash
yum install langpacks-zh_CN.noarch    # CentOS 8 安装
```

```bash
yum reinstall glibc-common
```

安装字体

```bash
yum groupinstall "fonts"
```

查看系统支持的语言（是否支持zh_CN.utf8）

```bash
locale -a | grep zh
```

编辑系统语言文件

```bash
vim /etc/locale.conf
LANG=zh_CN.utf8
```

立即生效

```bash
source /etc/locale.conf
```



也可以直接设置系统语言

```bash
localectl  set-locale LANG=zh_CN.utf8
```

