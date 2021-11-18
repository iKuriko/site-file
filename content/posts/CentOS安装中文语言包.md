---
title: "CentOS7修改中文"
date: 2021-11-05T10:34:28+08:00
draft: true
---

安装中文语言包

```bash
yum install kde-l10n-Chinese
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

```
source /etc/locale.conf
```

也可以直接设置系统语言

```bash
localectl  set-locale LANG=zh_CN.utf8
```

