---
title: "RHEL8 Packaging for OpenvSwitch"
date: 2021-11-23T16:25:50+08:00
draft: true
---

## Preparation

操作的系统环境：CentOS 8.4.2105

```bash
cat /etc/redhat-release 
CentOS Linux release 8.4.2105
```

``` bash
uname -r
4.18.0-348.el8.x86_64
```

Ovs软件包版本：openvswitch-2.16.1.tar.gz

```bash
wget https://www.openvswitch.org/releases/openvswitch-2.16.1.tar.gz
```

官网下载地址：https://www.openvswitch.org/releases/openvswitch-2.16.1.tar.gz



## Build Requirements

安装 RPM 工具和通用构建依赖项

```bash
dnf install @'Development Tools' rpm-build dnf-plugins-core
```

解压软件包

```bash
tar -zxvf openvswitch-2.16.1.tar.gz -C /usr/local/
cd openvswitch-2.16.1/
```

安装 Open vSwitch 特定的构建依赖项。依赖项在 SPEC 文件中列出，首先需要将 VERSION 标记替换为有效的 SPEC。

```bash
sed -e 's/@VERSION@/0.0.1/' rhel/openvswitch-fedora.spec.in  > /tmp/ovs.spec
```

启用powertools源，安装构建依赖的额外软件包

```bash
dnf --enablerepo=powertools install python3-sphinx groff
```

安装 Open vSwitch 特定的构建依赖项。

```bash
dnf builddep /tmp/ovs.spec
```



## Bootstraping

初始化，如果下载的是完整的tar包则不需要执行此命令，反之使用Git上的源代码来构建则需使用此命令以生成“configure”配置脚本

```bash
./boot.sh
```



## Configuring

进入软件包解压的目录

```bash
cd openvswitch-2.16.1/
```

使用默认配置[^1]

```bash
./configure
```



## Building

构建Open vSwitch 用户空间 RPM

```bash
make rpm-fedora
```

在openvswitch包中启用 DPDK 支持

```bash
dnf install dpdk-devel numactl-devel -y
```

```bash
dnf --enablerepo=powertools install libpcap-devel libbpf-devel
```

```bash
make rpm-fedora RPMBUILD_OPT="--with dpdk --without check"
```



在openvswitch包中启用 AF_XDP 支持

```bash
make rpm-fedora RPMBUILD_OPT="--with afxdp --without check"
```



让上述命令自动运行 Open vSwitch 单元测试（可选）

```bash
make rpm-fedora RPMBUILD_OPT="--with check"
```



为当前运行的内核版本构建Open vSwitch 内核模块

```bash
make rpm-fedora-kmod
```



为其他内核版本构建Open vSwitch 内核模块，可以通过kversion宏指定所需的内核版本

```bash
make rpm-fedora-kmod RPMBUILD_OPT='-D "kversion 4.3.4-300.fc23.x86_64"'
```



新生成的RPM包的位置

```bash
ls /usr/local/openvswitch/rpm/rpmbuild/RPMS/x86_64/
network-scripts-openvswitch-2.16.1-1.el8.x86_64.rpm  openvswitch-debugsource-2.16.1-1.el8.x86_64.rpm  openvswitch-kmod-2.16.1-1.el8.x86_64.rpm
openvswitch-2.16.1-1.el8.x86_64.rpm                  openvswitch-devel-2.16.1-1.el8.x86_64.rpm
openvswitch-debuginfo-2.16.1-1.el8.x86_64.rpm        openvswitch-ipsec-2.16.1-1.el8.x86_64.rpm
```

打包构建完成的RPM软件包

```bash
tar -zcvf openvswitch-2.16.1-centos8.tar.gz /usr/local/openvswitch/rpm/rpmbuild/RPMS/x86_64/
```



## Installing

安装软件包的步骤

```bash
tar -zxvf openvswitch-2.16.1-centos-8.tar.gz
```

```bash
cd openvswitch-2.16.1-centos-8/
```

```bash
yum localinstall openvswitch-2.16.1-1.el8.x86_64.rpm openvswitch-devel-2.16.1-1.el8.x86_64.rpm
```

```bash
systemctl start openvswitch
```

```bash
systemctl enable openvswitch
```



参考的文献：

[https://docs.openvswitch.org/en/latest/intro/install/fedora/?highlight=build%20openvswitch](https://docs.openvswitch.org/en/latest/intro/install/fedora/?highlight=build%20openvswitch)



[^1]: 一般情况下，使用默认配置即可，详情见官方文档：https://docs.openvswitch.org/en/latest/intro/install/general/#configuring
