---
title: "KVM 虚拟化"
date: 2021-11-10T16:36:54+08:00
draft: true
---

## 简介

KVM是开源软件，全称是kernel-based virtual machine（基于内核的虚拟机）。使用半虚拟化技术创建虚拟机的模块，可以将Linux内核转化为一个hypervisor[^1]。

KVM的前身是QEMU，2008年被Red Hat公司收购并获得的一项hypervisor技术，不过Red Hat的KVM被认为是将成为未来Linux hypervisor的主流，准确来说，KVM仅仅是Linux内核的一个模块，管理和创建完整的KVM虚拟机，需要更多的辅助工具。

KVM在2007年2月被整合到Linux 2.6.20核心中，以可加载核心模块的方式被移植到FreeBSD及illumos上，它依托CPU和虚拟化指令集（如Inter-VT、AMD-V）实现高性能的虚拟化支持；由于与Linux内核高度整合，因此在性能、安全性、兼容性、稳定性上都有很好的表现。

KVM环境中运行的每个虚拟化操作系统都将表现为单个独立的系统进程。因此它可以很方便地与Linux系统中的安全模块进行整合（SElinux），可以灵活地实现资源的管理及分配。



## 虚拟化类型

**全虚拟化**：虚拟机中运行的软件与系统不需经过任何修改，就好比运行在真实硬件一样；但依然使用虚拟硬件设备，并且需要CPU的虚拟化支持（如Intel VT技术或者AMD V技术)，是基于硬件的完全虚拟化，性能更好。



**半虚拟化**：另一种类似于全虚拟化的热门技术，同样使用 Hypervisor (虚拟机管理程序) 分享存取底层的硬件, 但是它的客户操作系统集成了虚拟化方面的代码，不需要CPU的硬件支持。该方法无需重新编译或引起陷阱，因为操作系统自身能够与虚拟进程进行很好的协作；但半虚拟化需要客户操作系统做一些修改（配合VDSM），这是一个不足之处，但是半虚拟化提供了与原始系统相近的性能，与全虚拟化一样，半虚拟化可以同时能支持多个不同的操作系统。





## 安装 KVM

以下操作的系统环境为CentOS 8.4.2105

```bash
cat /etc/redhat-release 
CentOS Linux release 8.4.2105
```

```bash
uname -r
4.18.0-305.3.1.el8.x86_64
```

先查看CPU是否支持虚拟化[^2]

```bash
cat /proc/cpuinfo | egrep 'vmx|svm'
```



关闭SELinux

```bash
sed -i "/^SELINUX/s/enforcing/disabled/g" /etc/selinux/config
```

 

安装KVM所需的软件包

```bash
yum -y install qemu-kvm libvirt libvirt-daemon libvirt-daemon-driver-qemu libvirt-client virt-manager libguestfs-tools virt-install virt-viewer virt-v2v bridge-utils
```

所需软件包详解

```
qemu-kvm                      # KVM模块
libvirt                       # 虚拟机管理工具包
libvirt-daemon                
libvirt-daemon-driver-qemu    
libvirt-client                
virt-manager                  # 图形界面管理虚拟机
libguestfs-tools              # virt-cat等命令的支持软件包
virt-install                  # 命令行安装虚机
virt-viewer                   # 图形化连接虚机
virt-v2v                      # 将其他平台的虚机转换为kvm虚机
bridge-utils                  # 网桥支持工具包
```





重启宿主机，以便加载 kvm 模块

```bash
reboot
```



查看KVM模块是否被正确加载

```bash
lsmod | grep kvm          #输出以下参数，说明KVM模块已经正确加载
kvm_intel       162153 0
kvm          525259 1 kvm_intel       
```

 

开启kvm服务，并且设置其开机自动启动

```bash
systemctl start libvirtd
```

```bash
systemctl enable libvirtd
```



查看状态，如Active: active (running)，说明运行情况良好

```bash
systemctl status libvirtd
```



## 配置 KVM网络

宿主服务器安装完成KVM，首先要设定网络，在libvirt中运行KVM网络有两种方法：NAT和Bridge，**默认的网络是NAT模式**



先删除默认的default网络[^3]

```bash
virsh net-destroy default
Network default destroyed
```

```bash
virsh net-undefine default
Network default has been undefined
```

重启服务

```bash
systemctl restart libvirtd
```

#按需求选择是否开启路由转发

```bash
vim /etc/sysctl.conf

net.ipv4.ip_forward = 1    #开启路由转发
```

```bash
sysctl -p                  #立即生效
```



### 配置 NAT 模式

宿主机网络配置

```bash
vim /etc/sysconfig/network-scripts/ifcfg-natbr0
```

```bash
DEVICE=natbr0
NAME=natbr0
ONBOOT=yes
BOOTPROTO=none
IPADDR=192.168.112.1
TYPE=Bridge
NETMASK=255.255.255.0
```

```bash
ip link set dev natbr0 up
```

新建网络定义 xml 文件


```bash
vim /tmp/natbr0.xml
```

```xml
<network>
 <name>nat</name>
 <bridge name="natbr0"/>
 <forward/>
 <ip address="192.168.112.1" netmask="255.255.255.0">
  <dhcp>
   <range start="192.168.112.2" end="192.168.112.254"/>
  </dhcp>
 </ip>
</network>
```



### 配置 Linux-Bridge 模式

宿主机网络配置

```bash
vim /etc/sysconfig/network-scripts/ifcfg-br1
```

```bash
DEVICE=br1
NAME=br1
ONBOOT=yes
BOOTPROTO=none
IPADDR=192.168.120.1
TYPE=Bridge
NETMASK=255.255.255.0
```

```bash
ip link set dev br1 up
```



新建网络定义 xml 文件

```bash
vim /tmp/br1.xml
```

```xml
<network>
  <name>br1</name>
  <forward mode="bridge"/>
  <bridge name="br1"/>
</network>
```







### 配置 Ovs-Bridge 模式

宿主机网络配置

使用ovs网桥需安装 openvswitch 软件包

```bash
ovs-vsctl add-br ovs-br1    #创建ovs网桥
```

新建网络定义 xml 文件

```bash
vim /tmp/ovs-br1.xml
```

```xml
<network>
  <name>ovs-br1</name>
  <forward mode="bridge"/>
  <bridge name="ovs-br1"/>
  <virtualport type="openvswitch"/>
</network>
```



### 定义 KVM网络

```bash
virsh net-define /tmp/br1.xml
Network BR1 defined from /tmp/br1.xml
```

启动网络

```bash
virsh net-start BR1
Network BR1 started
```

 自启动网络

```bash
virsh net-autostart BR1
Network BR1 marked as autostarted
```

 查看

```bash
virsh net-list
Name         State   Autostart   Persistent
----------------------------------------------------------
BR1         active   yes      yes
default       active   yes      yes
```



## 创建 KVM 虚机

安装前要设置环境语言为英文LANG="en_US.UTF-8"[^4]

```bash
vim /etc/locale.conf

LANG="en_US.UTF-8"
```



新建磁盘及镜像存储目录

```bash
mkdir -pv /kvm/{store,iso}
```

 

新建虚拟磁盘(下面会详细介绍该命令)

```bash
qemu-img create -f qcow2 /kvm/store/CentOS-7.9.qcow2 20G
```



给与镜像文件权限，特别注意.iso镜像文件一定放到/home 或者在根目录重新创建目录，不然会因为权限报错，无法创建虚拟机。

```bash
chown root:root /iso/CentOS-7-x86_64-Minimal-2009.iso 
```



使用 virt-install 创建虚拟机（也可以通过 virt-manager 图形化工具创建）

```bash
virt-install --virt-type=kvm --name=CentOS7.9 --os-type=linux --vcpus=1 --memory=1024 --location=/kvm/iso/CentOS-7-x86_64-Minimal-2009.iso --disk=/kvm/store/CentOS-7.9.qcow2,size=20,format=qcow2 --network bridge=br0,model=virtio --accelerate --autostart --extra-args='console=ttyS0'
```

virt-install 参数

```bash
--name             # 指定虚拟机名称
--memory           # 分配虚拟机内存大小，以 MB 为单位
--vcpus            # 分配CPU核心数，最大与实体机CPU核心数相同
--cpuset           # 设置哪个物理CPU能够被虚拟机使用
--location         # 指定安装介质路径，如光盘镜像或网络安装，有本地、nfs、http、ftp几种
--extra-args=EXTRA # 当执行从"--location"选项指定位置的客户机安装时，用于传递额外的内核命令行参数到安装程序根，例如指定kickstart文件的位置，--extra-args "ks=http://172.16.0.1/class.cfg"
--autostart        # 指定虚拟机是否在物理启动后自动启动
--cdrom            # 指定安装镜像iso(和上面的选项相同，不过这个特指iso镜像)
--pxe              # 基于PXE安装  
--livecd           # 把光盘当作LiveCD，直接启动系统
--boot cdrom,hd,network   # 指定引导次序
--disk             # 指定虚拟机的磁盘存储位置，size，以GB为单位，format，磁盘的格式。
--nodisks          # 不使用本地磁盘安装，在LiveCD模式中常用
--network          # 虚拟机使用的网络类型，其中子选项，bridge=br0 指定桥接网卡的名称。
--nonetworks       # 虚拟机不使用网络功能
--accelerate       # KVM或KQEMU内核加速,推荐最好加上。如果KVM和KQEMU都支持，KVM加速器优先使用。
--vnc              # 开启vnc远程管理
--vncport          # 指定VNC监控端口，默认端口为5900，端口不能重复。
--vnclisten        # 指定VNC绑定IP，默认绑定127.0.0.1，这里改为0.0.0.0。
--graphics         # 指定图形显示相关的配置，此选项不会配置任何显示硬件（如显卡），而是仅指定访问的接口
--nographics       # 指定没有控制台被分配给客户机
--video=VIDEO      # 指定显卡设备模型，可用取值为cirrus、vga、qxl或vmvga
--uuid             # 客户端UUID 默认不写时，系统会自动生成
--noautoconsole    # 禁止自动连接至虚拟机的控制台
--hvm              # 全虚拟化
--paravirt         # 半虚拟化
--force            # 禁止命令进入交互式模式，如果有需要回答yes或no选项，则自动回答为yes
--debug            # 显示debug信息
--host-device=HOSTDEV # 附加一个物理主机设备到客户机。HOSTDEV是随着libvirt使用的一个节点设备名（具体设备如’virsh nodedev-list’的显示的结果）
--os-type=OS_TYPE（linux,windows）            # 针对一类操作系统优化虚拟机配置（非必要）
--os-variant=OS_VARIANT（rhel7,winx7,win2k8） # 针对特定操作系统变体进一步优化虚拟机配置（非必要）
```

图形化控制台，服务器未安装图形化则无法开启，但可通过远程连接软件强制开启(不稳定)

```bash
virt-manager
```

查看镜像的信息

```bash
virt-filesystems --long --parts --blkdevs -h -a Centos_7.qcow2
```

kvm虚拟机的配置文件位置

```bash
ls /etc/libvirt/qemu/
```



## KVM 基本管理命令

```bash
virsh list                          # 查看已开启的虚拟机
virsh list --all                    # 查看所有虚拟机
virsh net-list                      # 查看已开启的虚拟网络
virsh net-list --all                # 查看所有虚拟网络
virsh net-define default.xml        # 定义kvm网络
virsh net-undefine default          # 删除kvm网络
virsh net-start default             # 启动kvm网络
virsh net-autostart default         # 自启动kvm网络
virsh domiflist Centos              # 查看虚机内的网卡
virsh dumpxml vm-name               # 查看kvm虚拟机配置文件
virsh start vm-name                 # 启动kvm虚拟机
virsh shutdown vm-name              # 关闭kvm虚拟机（正常关机）
virsh destroy vm-name               # 强制关闭虚拟机（非正常关机，相当于直接拔掉电源）
virsh undefine vm-name              # 移除定义kvm虚拟机（只会删除配置文件，不会删除磁盘文件）
virsh define file-name.xml          # 根据xml配置文件定义虚拟机
virsh suspend vm-name               # 挂起虚拟机
virsh resumed vm-name               # 恢复被挂起的虚拟机
virsh autostart vm-name             # 开机自启动kvm虚拟机
virsh console vm-name               # 使用console控制台连接虚拟机，键入 Ctrl+5 退出
virsh dumpxml vm-name > ~/vm.xml    #导出虚拟机的配置文件 
virsh setvcpus vm-name --maximum 4 --config   # 更改CPU
virsh setmaxmem vm-name 1024 --config         # 更改内存
virsh dominfo vm-name                         # 查看信息
```



## KVM 高级功能管理

### 磁盘格式转换

- KVM虚拟机常用磁盘格式为raw与qcow2格式，默认使用raw格式，那么其中raw格式的磁盘性能最好、速度最快，但不支持AES加密、zlib磁盘压缩等新功能，而qcow2格式磁盘存储空间更小，并支持AES加密、zlib、快照等新功能，缺点是性能较差
- 如果想管理指定虚拟机磁盘（如分区情况、磁盘数量等），可以使用"libguestfs-tools"工具（一般默认安装），下面举例，说明如和转换磁盘格式



查看指定磁盘文件的信息（如磁盘格式、占用磁盘大小等）

``` bash
qemu-img info /kvm/store/Centos.img
```



qemu-img 命令可以对kvm的磁盘镜像进行管理

``` bash
qemu-img convert -f raw -O qcow2 <raw格式磁盘镜像路径> <qcow2格式磁盘镜像路径>
```

```
选项：
-c：对输出的镜像文件进行压缩，但只有qcow2和qcow格式支持
-f：指定源磁盘格式
-O：指定转换后磁盘格式
```



将指定raw格式文件转换为qcow2磁盘格式文件（注意该虚拟机需关机）

``` bash
qemu-img convert -f raw -O qcow2 /kvm/store/Centos.img /kvm/store/Centos.qcow2
```

<font color=red>注意：转换后，需更改KVM虚拟机配置文件，因为虚拟机中还是用原磁盘格式文件，需要更改为新转换后的磁盘文件，才能使用新磁盘格式</font>



编辑指定名为Centos虚拟机配置文件

```  bash
virsh edit Centos
 <driver name='qemu' type='qcow2' cache='none'/>
 <source file='/kvm/store/Centos.qcow2'/>
```

启动虚机

```bash
virsh start Centos
```



### 管理虚机文件

**查看指定KVM虚拟机磁盘文件里指定路径内容**

``` bash
virt-cat -a {磁盘文件路径} {文件绝对路径}
```
栗：查看Centos.qcow2磁盘文件中/etc/sysconfig/network内容

```bash
virt-cat -a /kvm/store/Centos.qcow2 /etc/sysconfig/network
```

**编辑指定KVM虚拟机磁盘文件里指定路径内容**

``` bash
virt-edit -a {磁盘文件路径} {文件绝对路径}
```

栗：编辑Centos的/etc/sysconfig/network文件

```bash
virt-edit -a /kvm/store/Centos.qcow2 /etc/sysconfig/network
```

**查看指定KVM虚拟机磁盘使用情况**

``` bash
virt-df -h vm-name
```



### 虚机克隆和快照



#### 创建克隆

```bash
virt-clone -o CentOS7.9_01 -n test1 -f /kvm/store/test1.qcow2
```

virt-clone 参数

```bash
--version            # 查看版本
--help               # 查看帮助信息
--connect=URI        # 连接到虚拟机管理程序 libvirt 的URI
-o                   # 原始虚拟机名称 原始虚拟机名称，必须为关闭或者暂停状态
--name               # 新虚拟机名称
--auto-clone         # 从原来的虚拟机配置自动生成克隆名称和存储路径
--uuid=NEW_UUID      # 克隆虚拟机的新的UUID，默认值是一个随机生成的UUID
--mac=NEW_MAC        # 设置一个新的mac地址，默认为随机生成 MAC
--file=NEW_DISKFILE  # 为新客户机使用新的磁盘镜像文件地址。
--force-copy=TARGET  # 强制复制设备。
--nonsparse          # 不使用稀疏文件复制磁盘映像。
```



#### 创建快照

```bash
virsh snapshot-create test1
Domain snapshot 1624247774 created
```

```bash
virsh snapshot-list test1

Name         Creation Time       State
------------------------------------------------------------
1624247774      2021-06-21 11:56:14 +0800 shutoff
```

恢复快照

```bash
virsh snapshot-list test1

Name         Creation Time       State
------------------------------------------------------------
1624249154      2021-06-21 12:19:14 +0800 shutoff
```

```bash
virsh snapshot-revert test1 1624249154
```

删除快照

```bash
virsh snapshot-list test3

Name         Creation Time       State
------------------------------------------------------------
1624249270      2021-06-21 12:21:10 +0800 shutoff
1624249928      2021-06-21 12:32:08 +0800 shutoff
```

```bash
virsh snapshot-delete test3 1624249928
Domain snapshot 1624249928 deleted
```

```bash
virsh snapshot-list test3

Name         Creation Time       State
------------------------------------------------------------
1624249270      2021-06-21 12:21:10 +0800 shutoff
```



## Error 随笔

添加 ovs-br 后启动虚拟机时报错

```bash
virsh start --domain test2
error: Failed to start domain test2
error: Unable to add bridge br0 port vnet2: Operation not supported
```



启动虚拟机时 libvirt 会尝试 linux 默认的LinuxBridge，而不是openvswitch

```bash
virsh edit test1    #编辑虚拟机配置文件，在<interface>加入<virtualport type='openvswitch'/>
```

```xml
<interface type='bridge'>

 <virtualport type='openvswitch'/>

</interface>
```

---



[^1]: Hypervisor，又称"虚拟机监视器"（virtual machine monitor，缩写为 VMM），是用来建立与执行[虚拟机器](https://baike.baidu.com/item/虚拟机器)的软件、固件或硬件。
[^2]: KVM 是基于 x86 虚拟化扩展(Intel VT 或者 AMD-V) 技术的虚拟机软件，所以查看 CPU 是否支持 VT 技术，就可以判断是否支持KVM。有返回结果，如果结果中有vmx（Intel）或svm(AMD)字样，就说明CPU的支持的。
[^3]:为了方便管理KVM网络，删除自带的默认网络，默认的网络模式为NAT模式 
[^4]:  如果是中文的话某些版本可能会报错或乱码。在CentOS 7/8 中修改 /etc/locale.conf
