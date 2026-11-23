---
title: "Rocky Linux 8.10 打包 OpenvSwitch"
date: 2026-11-23T10:15:22+08:00
draft: true
tags:
  - OpenvSwitch
description: 在 Rocky Linux 8.10 上构建 OpenvSwitch 软件包
---

# 实战：Rocky Linux 8.10 打包 OpenvSwitch 3.7.1 RPM

> **环境元数据**
>
> - **操作系统**：Rocky Linux 8.10 (Green Obsidian)
> - **内核版本**：`4.18.0-553.184.1.el8_10.x86_64`
> - **目标软件**：Open vSwitch (OVS) 3.7.1
> - **打包产物**：Userspace RPM 包 (`openvswitch`, `openvswitch-devel` 等)

------

## 1. 背景与前置说明

在生产环境或私有云集群网络（如 SDN、KVM 虚拟交换机、容器网络网桥）中，发行版官方仓库提供的 Open vSwitch 版本往往滞后。为了获取更新的特性与性能修复，通常需要在统一的 CI/构建节点上基于源码构建适配特定操作系统内核与依赖的 RPM 包，随后打归档分发到生产节点。

> **提示**：从 CentOS/RHEL 8 开始，OVS 运行所需的内核模块 `openvswitch.ko` 已经内建于系统提供的 `kernel-modules-extra` 包中，通常无需编译独立的 OVS 内核驱动包（kmod-openvswitch），直接编译构建用户态管理与核心服务程序（Userspace RPM）即可。

------

## 2. 构建环境准备

### 2.1 启用必要软件源与工具链

Rocky Linux 8 构建 RPM 包需要启用 `powertools` 仓库以拉取部分底层依赖库：

安装基础包管理工具与 EPEL 源 
``` bash
dnf install -y dnf-plugins-core epel-release 
```

启用 PowerTools 仓库 (CRB) 
```bash
dnf config-manager --set-enabled powertools
```
安装 GCC、Make 等开发工具套件 
```bash
dnf groupinstall -y "Development Tools" # 4. 安装 RPM 构建相关工具 dnf install -y rpm-build rpmdevtools
```
验证基础编译工具链版本：
```bash
gcc --version make --version rpmbuild --version
```
------

## 3. 源码解压与依赖解析

从官网或官方发布仓库下载好 `openvswitch-3.7.1.tar.gz` 后，放置于工作目录（如 `/root`）：
```bash
cd /root tar -xzf openvswitch-3.7.1.tar.gz cd /root/openvswitch-3.7.1
```

### 3.1 提取并生成 SPEC 文件

OVS 源码自带了针对 Fedora/RHEL 系列的 spec 模板文件 `rhel/openvswitch-fedora.spec.in`。我们需要将模板中的版本占位符替换为当前版本：
```bash
sed 's/@VERSION@/3.7.1/' rhel/openvswitch-fedora.spec.in > /tmp/ovs.spec
```
### 3.2 自动安装所有编译依赖 (`dnf builddep`)

利用 `dnf builddep` 命令，系统会自动分析 `/tmp/ovs.spec` 中声明的 `BuildRequires`，并从已配置的仓库中一次性自动补齐所有依赖包：
```bash
dnf builddep -y /tmp/ovs.spec
```
------

## 4. 预配置与 RPM 制作

### 4.1 执行 Configure 配置

指定标准系统目录前缀进行配置，同时检查 Python3 环境绑定：
```bash
./configure \     --prefix=/usr \     --sysconfdir=/etc \     --localstatedir=/var
```
确认 Makefile 中绑定正确的 Python3 解释器路径 
```bash
grep '^PYTHON3[[:space:]]*=' Makefile
```
### 4.2 编译源码与打包 RPM

执行 `make rpm-fedora` 指令开始自动化构建 RPM。为了加快构建速度并跳过耗时漫长的单元测试套件，建议附加 `RPMBUILD_OPT="--without check"`：

```bash
# 预编译核心二进制 
make -j"$(nproc)" 
# 构建 Fedora/RHEL 系列 RPM 包（跳过单元测试） 
make rpm-fedora RPMBUILD_OPT="--without check"`
```
构建成功后，生成的 RPM 文件通常位于源码目录下的 `rpm/rpmbuild/RPMS/`：
```bash
ls -lh /root/openvswitch-3.7.1/rpm/rpmbuild/RPMS/x86_64/ 
ls -lh /root/openvswitch-3.7.1/rpm/rpmbuild/RPMS/noarch/
```
------

## 5. 归档与版本化归类

为了便于在产线与特定内核版本镜像之间做精准回溯，建议以系统内核版本与架构为命名规范建立归档目录：
```bash
KERNEL_VER="$(uname -r)" 
PACKAGE_DIR="openvswitch-3.7.1-1.el8.x86_64-${KERNEL_VER}" 

mkdir -p ~/"${PACKAGE_DIR}" 

# 收集所有架构及平台通用 RPM 
cp /root/openvswitch-3.7.1/rpm/rpmbuild/RPMS/x86_64/* ~/"${PACKAGE_DIR}/" cp /root/openvswitch-3.7.1/rpm/rpmbuild/RPMS/noarch/* ~/"${PACKAGE_DIR}/" 

# 打包归档为 tar.gz 
cd ~ tar -zcvf "${PACKAGE_DIR}.tar.gz" "${PACKAGE_DIR}/"`
```
------

## 6. 内核依赖校验与本地安装验证

在目标机器安装之前，必须确认目标主机的内核模块完备性与服务自启。

### 6.1 检查 Open vSwitch 内核模块

确认当前内核是否具备 `openvswitch.ko`：
```bash
# 检查模块信息 
modinfo openvswitch 

# 查找驱动模块文件路径 
find "/usr/lib/modules/$(uname -r)" -type f -name 'openvswitch.ko*' 

# 确保安装了 kernel-modules-extra 
rpm -q "kernel-modules-extra-$(uname -r)"
```
> **排坑说明**：如果找不到 `openvswitch.ko`，直接执行 `dnf install -y kernel-modules-extra-$(uname -r)` 补齐即可。

### 6.2 本地安装与启动验证

进入归档包目录进行本地 RPM 安装与冒烟测试：
```bash
cd ~/"${PACKAGE_DIR}" 
# 安装主程序包与开发依赖 
dnf localinstall -y openvswitch-3.7.1-1.el8.x86_64.rpm openvswitch-devel-3.7.1-1.el8.x86_64.rpm 

# 启动并激活 OVS 服务 
systemctl start openvswitch 
systemctl enable openvswitch 

# 校验服务状态与数据库拓扑 
ovs-vsctl show
```
若能看到类似如下输出，说明 OVS 核心守护进程（`ovs-vswitchd` 与 `ovsdb-server`）已正常运行并就绪：
```bash
c6168e21-5a52-4f32-84da-570a9dd37a54     
ovs_version: "3.7.1"
```
------

## 7. 踩坑与经验总结 (Caveats)

1. **PowerTools 仓库必须启用**：RHEL/Rocky 8 下许多编译期依赖（如特定版本的 libunwind、groff、graphviz 等）移至了 PowerTools/CRB 仓库，如果不开启会导致 `dnf builddep` 报无法找到依赖。
2. `--without check` 的权衡：OVS 源码自带的大量网络虚拟环境测试（OVS sandbox、veth 隧道互通）极其消耗时间（通常需要 20~40 分钟以上）。打包确认无误后跳过测试可以极大提升交付效率。
3. **内核版本匹配**：虽然 Userspace RPM 包具备较好的跨小版本兼容性，但在跨大版本升级系统内核后，仍建议检查并回归测试 `openvswitch.ko` 模块与用户态程序在流表匹配时的接口兼容性。

