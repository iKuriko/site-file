---
title: "CentOS8 AP"
date: 2026-06-17T16:02:06+08:00
draft: true
tags:
  - Services
description: Linux 化身 WIFI 路由器
---



## 安装软件包

查看是否有无线网卡

```bash
lspci | grep Wireless 
```

安装依赖软件包

```bash
dnf install NetworkManager-wifi iw
```

```bash
systemctl restart NetworkManager
```

查看内核模块是否加载（不同无线网卡模块名称可能不同，使用`lspci`查看）

```bash
lsmod | grep rtl8188ee
```

 

若无输出，手动加载模块

```bash
modprobe rtl8188ee
```



## 配置服务

开启WiFi

```bash
nmcli radio wifi on 
```

 

查看wifi设备是否是disconnected状态

```bash
nmcli device 
```

 

如果是unmanaged，则执行此命令

```bash
nmcli dev set wlp7s0 managed yes
```

 

扫描附件的WIFI

```bash
nmcli device wifi list 
```

 

新建br0网桥

```bash
nmcli connection add type bridge con-name br0 ifname br0 bridge.stp no bridge.multicast-snooping no ipv4.addresses 172.16.10.1/24 ipv4.method manual ipv6.method disabled
```

```bash
nmcli connection up br0
```

 

命令行配置WiFi

```bash
nmcli device wifi hotspot ifname wlp7s0 con-name freewifi ssid freewifi
```

```bash
nmcli connection modify freewifi ipv4.method disabled autoconnect yes master br0
```

```bash
nmtui    #选择freewifi添加wifi密码
```



## 启动热点

```bash
nmcli connection up freewifi
```



## 驱动问题解决

更新驱动

```bash
yum -y localinstall iwl7260-firmware-25.30.13.0-127.el8_10.1.noarch.rpm
```

 

安装缺失的固件包（如有必要）

```bash
dnf install linux-firmware
reboot
```



解除RFKill锁定，检查无线设备是否被禁用

```bash
rfkill list
```

 

若显示Soft blocked: yes，解除锁定

```bash
sudo rfkill unblock wifi
```

 

 

如果还是识别不了网卡，编译安装驱动

启用ELRepo仓库

```bash
sudo dnf install https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
```

 

安装DKMS和编译工具

```bash
sudo dnf install dkms kernel-devel-$(uname -r) gcc
```

 

从源码编译驱动

```bash
git clone https://github.com/lwfinger/rtlwifi_new.git

cd rtlwifi_new

make -j4

sudo make install

sudo modprobe rtl8188ee
```



永久加载驱动

```bash
echo "rtl8188ee" | sudo tee /etc/modules-load.d/rtl8188ee.conf
```

 

 

常见问题排查：



如果提示`Operation not possible due to RF-kill`：检查物理无线开关

 

持续显示unmanaged：编辑`/etc/NetworkManager/NetworkManager.conf`，在`[ifupdown]`段设置`managed=true`后重启服务

 

信号弱/不稳定：尝试调整天线位置，或通过`iwconfig wlp7s0 power off`关闭省电模式