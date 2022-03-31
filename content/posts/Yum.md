---
title: "Yum"
date: 2022-03-30T14:48:31+08:00
draft: true
tags:
  - CentOS Linux
description: YUM 是基于RPM包的软件包管理工具，能自动解决 RPM 包复杂的依赖关系，并且一次性安装所有依赖的软件包
---



## 简介

YUM（Yellow dog Updater，Modified），由 Yellow Dog Linux发行版的开发者 Terra Soft 开发，使用python编写，后经杜克大学linux开发团队改进。yum是一个在Fedora、RedHat，以及SUSE中的shell前端软件包管理器。

 

YUM是基于RPM包的软件包管理工具，会自动解决 rpm 包的依赖关系，并且一次性安装所有依赖的软件包，安装、升级或是移除rpm包，收集rpm包相关信息。

 

YUM需要软件仓库（repostitory）的支撑，软件仓库是网络的（httpd,ftp）或是本地的（local），必须包含rpm包的头部信息（header），header包括了rpm包的各种信息（描述，功能，提供的文件，性能的需求，依赖关系等）。yum需要收集这些header并加以分析，才能自动化的完成余下的任务。

 

YUM可以同时配置多个软件仓库。它的全局配置文件在：/etc/yum.conf，仓库配置文件在：/etc/yum.repos.d/

 

## 配置



yum的配置文件分为两部分：主配置文件\<main>和仓库配置文件\<repository>

 

- Main 定义了全局配置选项，整个yum配置文件只有一个main（/etc/yum.conf）

- Repostiory 定义了每个软件仓库的具体配置，可以一到多个（/etc/yum.repos.d/）

 

### 全局配置

```bash
vim /etc/yum.conf    #修改全局配置文件
exclude=selinux*    #排除某些软件包在升级名单之外，可以用通配符，各个项目需要用空格隔开
gpgcheck=1    #设置是否开启gpg（GNU Private Guard）校验，1代表开启，0代表关闭，确保rpm包的来源是安全有效的，这里为全局设置，默认为0不开启
```

```bash
ll /etc/yum.repos.d/    #查看软件仓库配置文件

-rw-r--r--  1 root root 2523 7月   5 2021 CentOS-Base.repo    #网络源
-rw-r--r--  1 root root  630 7月   5 2021 CentOS-Media.repo    #本地源
-rw-r--r--. 1 root root  664 7月   5 2021 epel.repo    #扩展源
……
```



### 仓库配置

配置本地yum源

```bash
[local]
name=[local]    #本地仓库的名称，支持变量
baseurl=file:///mnt    #本地仓库的地址（一般为挂载的位置），其中url支持的协议有http://、ftp://、file://三种,可以挂载多个
gpgcheck=0    #不开启签名检查，1为开启签名检查
enabled=1    #启用本地仓库
gpgkey=file:///mnt/RPM-GPG-KEY-CentOS-8    #签名认证信息位置，gpgcheck为0时不需要此参数
```

```bash
mount /dev/cdrom /mnt
```



配置在线yum源

```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak    #备份原文件，以防万一
```

```bash
wget -O /etc/yum.repos.d/CentOS-Linux-BaseOS.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo    #将aliyun的在线源另存到本地
```

#如果没有安装wget命令，可以使用curl下载，效果相同

```bash
curl -o /etc/yum.repos.d/CentOS-Linux-BaseOS.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo 
```

```bash
yum clean all    #清理缓存
```

```bash
yum makecache    #生成缓存
```

 



## 常用命令

### 列表查看

```bash
yum repolist    #列出当前yum源中软件包个数

yum list    #列出软件仓库中所有可以安装或更新的rpm包

yum grouplist    #列出软件仓库中所有的软件包组

yum list htt*    #列出软件仓库中所有以htt开头的软件包，可以使用通配符

yum list updates    #列出软件仓库中所有可以更新的rpm包

yum list installed    #列出所有已经安装的rpm包

yum list extras    #列出已经安装，但是不包含在软件仓库中的rpm包
```

### 安装和卸载

```bash
yum install httpd    #安装httpd软件包，需要按y进行安装确认，加-y选项则跳过确认

yum remove httpd|yum erase httpd    #移除httpd软件包，包括与该包存在依赖关系的软件包

yum groupremove  xxx    #删除软件仓库中指定的软件包组
```

### 更新软件

```bash
yum check-update    #查看可更新的rpm软件包

yum update    #更新系统中所有的rpm软件包

yum groupupdate    #更新软件仓库中指定的软件包组

yum update kernel kernel-source    #更新指定的rpm软件包，比如 kernel 和 kernel-source

yum upgrade    #大规模的版本升级，包括内核，旧版本软件
```

### 清除缓存

```bash
yum clean packages    #清除缓存中的rpm包文件

yum clean headers    #清除缓存中的rpm包头部文件

yum clean oldheaders    #清除缓存中旧的rpm包头部文件

yum clean all    #清理所有缓存
```

### 软件包信息

```bash
yum info    #列出软件仓库中所有可以安装或更新的rpm包的详细信息

yum groupinfo  xxx    #列出软件仓库中指定软件包组的信息

yum info htt*    #列出软件仓库中所有以htt开头的软件包信息，可以使用通配符

yum info updates    #列出资源库中所有可以更新的rpm包的信息

yum info installed    #列出已经安装的所有的rpm包的信息

yum info extras    #列出已经安装，但不是包含在软件仓库中的rpm包信息
```

### 搜索软件

```bash
yum search httpd    #搜索指定名称的rpm包,可以使用通配符

yum provides httpd    #搜索包含指定文件名的rpm包
```



## RPM包管理

语法： 

```bash
rpm [选项] <rpm包名>
```

选项：

```bash
-i：安装新的rpm包
-e：卸载指定名称的rpm包
-U：检查更新某个软件包，若未安装，等同于-i选项
-F：检测更新某个软件包，若未安装，则放弃安装
-h：在安装或升级过程中，显示安装进度
-v：显示安装过程中的详细信息
--force：强制安装某个软件包
--nodeps：在安装、更新、卸载一个软件包时，无视依赖关系
-ql：软件包安装的文件路径
-qi：软件包的详细信息
-qa：列出所有已安装的软件包
-qf：查询文件所属的软件包
```

使用：

```bash
rpm --initdb    #初始化RPM数据库
```

```bash
rpm --rebuilddb    #重建RPM数据库
```

```bash
rpm --import /mnt/RPM-GPG-KEY-CentOS-7    #向rpm数据库导入公钥文件
```

```bash
rpm -ivh <rpm包名>    #安装软件包
```

```bash
rpm -e <rpm报名>    #卸载软件包
```

查询命令是由哪个软件包提供的

```bash
which vim    #查看命令的所在目录
/usr/bin/vim
```

```bash
rpm -qf /usr/bin/vim    #查看该目录是归属于哪一个软件包
vim-enhanced-7.4.629-8.el7_9.x86_64
```



