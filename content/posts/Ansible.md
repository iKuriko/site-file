---
title: "Ansible"
date: 2021-10-29T11:50:51+08:00
draft: true
tags:
  - CentOS Linux
  - Linux services
description: Ansible 是目前运维自动化工具中最简单、容易上手的一款优秀软件，能够用来管理各种资源，批量部署应用程序等。
---



## 简介

**Ansible**是目前**运维自动化工具**中最简单、容易上手的一款优秀软件，能够用来管理各种资源，批量部署应用程序等。能够帮助运维人员提高工作效率，减少人为失误。例如，借助于Ansible，我们可以轻松地对服务器进行初始化配置、安全基线配置，以及进行更新和打补丁操作。



**基于模块**：Ansible服务本身并没有批量部署的功能，它仅仅是一个框架，真正具有批量部署能力的是其所运行的模块。Ansible有上千个功能丰富且实用的模块，几乎可以满足一切需求，使用起来也非常简单，而且有详尽的帮助信息可供查阅，可以轻松上手。如果需要更高级的功能，也可以运用Python语言对Ansible进行二次开发。



**基于SSH**：相较于Chef、Puppet、SaltStack等C/S（客户端/服务器）架构的自动化工具来讲，尽管Ansible的性能并不是最好的，但由于它基于SSH远程会话协议，不需要客户端程序，只要知道受管主机的账号密码，就能直接用SSH协议进行远程控制，因此使用起来优势明显。由于受控节点不需要安装客户端，而SSH协议是Linux系统的标配，因此可以直接通过SSH协议进行远程控制。在控制节点上，也不用每次都重复开启服务程序，使用ansible命令直接调用模块进行控制即可。

 

Ansible服务专用术语

| 术语　|   服务名称 　　| 含义                                                        |
| :------------ | :-------------- | ------------------------------------------------------ |
| Control node  | 控制节点   | 指的是安装了Ansible服务的主机，也被称为Ansible控制端，主要是用来发布运行任务、调用功能模块，对其他主机进行批量控制。 |
| Managed nodes | 受控节点   | 指的是被Ansible服务所管理的主机，也被称为受控主机或客户端，是模块命令的被执行对象。 |
| Inventory     | 主机清单   | 指的是受控节点的列表，可以是IP地址、主机名称或者域名。       |
| Modules       | 模块       | 指的是上文提到的特定功能代码，默认自带有上千款功能模块，在Ansible Galaxy有超多可供选择。 |
| Task          | 任务       | 指的是Ansible客户端上面要被执行的操作。                      |
| Playbook      | 剧本       | 指的是通过YAML语言编写的可重复执行的任务列表，把常做的操作写入到剧本文件中，下次可以直接重复执行一遍。 |
| Roles         | 角色       | 从Ansible 1.2版本开始引入的新特性，用于结构化的组织Playbook，通过调用角色实现一连串的功能。 |





## 部署服务

 

备份原官方在线源

```bash
mv /etc/yum.repos.d/CentOS-Linux-BaseOS.repo /etc/yum.repos.d/CentOS-Linux-BaseOS.repo.bak
```

下载阿里云的在线源

```bash
curl -o /etc/yum.repos.d/CentOS-Linux-BaseOS.repo https://mirrors.aliyun.com/repo/Centos-8.repo
```

安装EPEL[^1]扩展软件包在线源

```bash
dnf install epel-release
```

安装ansible软件包

```bash
dnf install -y ansible
```

安装完毕后，Ansible服务便默认已经启动。可以看到Ansible服务的版本及配置信息。

```bash
ansible --version
```

Ansible服务配置文件优先级顺序[^2]

| 优先级 | 文件位置                 |
| ------ | ------------------------ |
| 高     | ./ansible.cfg            |
| 中     | ~/.ansible.cfg           |
| 低     | /etc/ansible/ansible.cfg |

 

 

## 设置主机清单（hosts）

 

> 主机清单（inventory）。将要管理的主机IP预写入`/etc/ansible/hosts`文件，这样后续执行ansible命令的任务时就包含这些主机了



受管主机信息如下

| 操作系统 | IP地址         | 功能用途 |
| -------- | -------------- | -------- |
| RHEL 8   | 192.168.88.130 | dev      |
| RHEL 8   | 192.168.88.139 | web      |

 

#由于主机清单文件/etc/ansible/hosts中默认存在大量的注释信息，建议先备份文件，然后新建一个文件

 

备份hosts文件

```bash
mv /etc/ansible/hosts /etc/ansible/hosts.bak
```

新建hosts文件

```bash
vim /etc/ansible/hosts

[devserver]
192.168.88.130
[webserver]
192.168.88.139 
```



Ansible是基于SSH实现自动化控制的，sshd服务在初次连接时回要求用户接受一次对方主机的指纹信息，然后再输入主机的账号和密码，这样是非常麻烦的。而Ansible提供了变量来解决这个问题



 Ansible常用变量汇总

| 参数               | 作用          |
| ------------------ | ------------- |
| ansible_ssh_host   | 受管主机名    |
| ansible_ssh_port   | 端口号        |
| ansible_ssh_user   | 默认账号      |
| ansible_ssh_pass   | 默认密码      |
| ansible_shell_type | Shell终端类型 |

 

用户只需要将对应的变量及信息填写到主机清单文件中，在执行任务时便会自动对账号和密码进行匹配，而不用每次重复输入它们。

```bash
vim /etc/ansible/hosts

[devserver]
192.168.88.130
[webserver]
192.168.88.139

[all:vars]
ansible_user=root
ansible_password=123
```

以结构化的方式显示出受管主机的信息

```bash
ansible-inventory --graph

@all:
 |--@devserver:
 | |--192.168.88.130
 |--@ungrouped:
 |--@webserver:
 | |--192.168.88.139
```

修改Ansible主配置文件

```bash
vim /etc/ansible/ansible.cfg

71 host_key_checking = False           #关闭SSH协议的指纹验证
107 remote_user = root                 #默认执行剧本时所使用的管理员名称为root
```

修改后不需要重启服务



> 连接host主机时ssh密钥不匹配怎么办

1. 在/root/.ssh/known_hosts中添加正确的主机密钥。

2. 修改在/root/.ssh/known_主机中有问题的ECDSA密钥：1

3. 直接删除 rm /root/.ssh/known_hosts

 

## 运行临时命令（ansible）

> Ansible服务的强大之处在于只需要一条命令，便可以操控成千上万台的主机节点，而ansible命令便是最得力的工具之一。ansible命令是用于执行临时任务的命令，一次性执行后结束，无法重复执行。前面提到，Ansible服务实际上只是一个框架，能够完成工作的是模块化功能代码。



Ansible服务常用模块名称及作用

| 模块名称       | 模块作用                                 |
| -------------- | ---------------------------------------- |
| ping           | 检查受管节点主机网络是否能够联通。       |
| yum            | 安装、更新及卸载软件包。                 |
| yum_repository | 管理主机的软件仓库配置文件。             |
| template       | 复制模板文件到受管节点主机。             |
| copy           | 新建、修改及复制文件。                   |
| user           | 创建、修改及删除用户。                   |
| group          | 创建、修改及删除用户组。                 |
| service        | 启动、关闭及查看服务状态。               |
| get_url        | 从网络中下载文件。                       |
| file           | 设置文件权限及创建快捷方式。             |
| cron           | 添加、修改及删除计划任务。               |
| command        | 直接执行用户指定的命令。                 |
| shell          | 直接执行用户指定的命令（支持特殊字符）。 |
| debug          | 输出调试或报错信息。                     |
| mount          | 挂载硬盘设备文件。                       |
| filesystem     | 格式化硬盘设备文件。                     |
| lineinfile     | 通过正则表达式修改文件内容。             |
| setup          | 收集受管节点主机上的系统及变量信息。     |
| firewalld      | 添加、修改及删除防火墙策略。             |
| lvg            | 管理主机的物理卷及卷组设备。             |
| lvol           | 管理主机的逻辑卷设备。                   |



在使用ansible命令时，必须指明受管主机的信息，如果已经设置过主机清单文件（/etc/ansible/hosts），则可以使用all参数来指代全体受管主机，或是用dev、test等主机组名称来指代某一组的主机。

 

ansible命令常用的语法格式为　ansible 受管主机节点 -m 模块名称 [-a模块参数]　，常见的参数如表所示。其中，-a是要传递给模块的参数，只有功能极其简单的模块才不需要额外参数，所以大多情况下 -m 与 -a 参数都会同时出现。

 

ansible命令常用参数

| 参数      | 作用                    |
| --------- | ----------------------- |
| -k        | 手动输入SSH协议密码     |
| -i        | 指定主机清单文件        |
| -m        | 指定要使用的模块名      |
| -M        | 指定要使用的模块路径    |
| -S        | 使用su命令              |
| -T        | 设置SSH协议连接超时时间 |
| -a        | 设置传递给模块的参数    |
| --version | 查看版本信息            |

如果想实现某个功能，但是却不知道用什么模块，又或者是知道了模块名称，但不清楚模块具体的作用，则建议使用ansible-doc命令进行查找。例如，列举出当前Ansible服务所支持的所有模块信息：

 

```bash
ansible-doc -l

a10_server                          Manage A10 Netwo...
a10_server_axapi3                       Manage A10 Netwo...
a10_service_group                       Manage A10 Netwo...
a10_virtual_server                      Manage A10 Netwo...
aci_aaa_user                         Manage AAA users...
aci_aaa_user_certificate                   Manage AAA user ...
aci_access_port_block_to_access_port             Manage port bloc...
aci_access_port_to_interface_policy_leaf_profile       Manage Fabric in...
aci_access_sub_port_block_to_access_port           Manage sub port ...
aci_aep                            Manage attachabl...
aci_aep_to_domain                       Bind AEPs to Phy...
aci_ap                            Manage top level...
aci_bd                            Manage Bridge Do...
aci_bd_subnet                         Manage Subnets (...
aci_bd_to_l3out                        Bind Bridge Doma...
aci_config_rollback                      Provides rollbac...
…………
```

 

一般情况下，很难通过名称来判别一个模块的作用，要么是参考模块后面的介绍信息，要么是平时多学多练，进行积累。例如，接下来随机查看一个模块的详细信息。ansible-doc命令会在屏幕上显示出这个模块的作用、可用参数及实例等信息：



```yaml
ansible-doc aci_domain

> ACI_DOMAIN  (/usr/lib/python3.6/site-packages/ansible/modules/network/aci/aci_>

​    Manage physical, virtual, bridged, routed or FC domain
​    profiles on Cisco ACI fabrics.

 * This module is maintained by an Ansible Partner

…………
```



之前已经成功地将受管主机的IP地址填写到主机清单文件中，接下来检查一下这些主机的网络连通性。ping模块用于进行简单的网络测试（类似于常用的ping命令）。可以使用ansible命令直接针对所有主机调用ping模块，不需要增加额外的参数，返回值若为SUCCESS，则表示主机当前在线。

 

```bash
ansible all -m ping

192.168.88.139 | SUCCESS => {
  "ansible_facts": {
​    "discovered_interpreter_python": "/usr/libexec/platform-python"
  },
  "changed": false,
  "ping": "pong"
}
192.168.88.130 | SUCCESS => {
  "ansible_facts": {
​    "discovered_interpreter_python": "/usr/libexec/platform-python"
  },
  "changed": false,
  "ping": "pong"
}
```



除了使用-m参数直接指定模块名称之外，还可以用-a参数将参数传递给模块，让模块的功能更高级，更好地满足当前生产的需求。例如，yum_repository模块的作用是管理主机的软件仓库，能够添加、修改及删除软件仓库的配置信息，参数相对比较复杂。遇到这种情况时，建议先用ansible-doc命令对其进行了解。尤其是下面的EXAMPLES结构段会有该模块的实例，对用户来说有非常高的参考价值。



```bash
ansible-doc -l | grep yum

yum                              Manages packages with the `yum' package manager                  
yum_repository                        Add or remove YUM repositories
```

```yaml
ansible-doc yum_repository

> YUM_REPOSITORY  (/usr/lib/python3.6/site-packages/ansible/modules/packaging/os/yum_repository.py)

     Add or remove YUM repositories in RPM-based Linux distributions. If you wish to update an existing repository
     definition use [ini_file] instead.
     
* This module is maintained by The Ansible Core Team

OPTIONS (= is mandatory):

EXAMPLES:

- name: Add repository
  yum_repository:
  name: epel
  description: EPEL YUM repo
  baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/

- name: Add multiple repositories into the same file (1/2)
  yum_repository:
  name: epel
  description: EPEL YUM repo
  file: external_repos
  baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/
  gpgcheck: no

- name: Add multiple repositories into the same file (2/2)
  yum_repository:
  name: rpmforge
  description: RPMforge YUM repo
  file: external_repos
  baseurl: http://apt.sw.be/redhat/el7/en/$basearch/rpmforge
```



参数并不是很多，而且与/etc/yum.repos.d/目录中的配置文件基本相似。现在，想为主机清单中的所有服务器新增一个如表所示的软件仓库，该怎么操作呢？



新增软件仓库信息

| 仓库名称    | CentOS-8-Base-mirrors.aliyun.com                             |
| ----------- | ------------------------------------------------------------ |
| 仓库描述    | mirrors.163.com                                              |
| 仓库地址    | https://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/ |
|             | http://mirrors.cloud.aliyuncs.com/centos/$releasever/BaseOS/$basearch/os/ |
|             | http://mirrors.aliyuncs.com/centos/$releasever/BaseOS/$basearch/os/ |
| GPG签名     | 启用                                                         |
| GPG密钥文件 | [https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official](file://media/cdrom/RPM-GPG-KEY-redhat-release) |

我们可以对照着EXAMPLE实例段，逐一对应填写需求值和参数，其标准格式是在-a参数后接整体参数（用单引号圈起），而各个参数字段的值则用双引号圈起。这是最严谨的写法。在执行下述命令后如果出现CHANGED字样，则表示修改已经成功：

 

```yaml
ansible all -m yum_repository -a 'name="CentOS-8-Base-mirrors.aliyun.com" description="mirrors.163.com" baseurl="https://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/ \n http://mirrors.aliyuncs.com/centos/$releasever/BaseOS/$basearch/os/ \n http://mirrors.cloud.aliyuncs.com/centos/$releasever/BaseOS/$basearch/os/" gpgcheck=yes enabled=1 gpgkey="https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official"'
```

```bash
192.168.88.139 | CHANGED => {
  "ansible_facts": {
​    "discovered_interpreter_python": "/usr/libexec/platform-python"
  },
  "changed": true,
  "repo": "CentOS-8-Base-mirrors.aliyun.com",
  "state": "present"
}

192.168.88.130 | CHANGED => {
  "ansible_facts": {
​    "discovered_interpreter_python": "/usr/libexec/platform-python"
  },
  "changed": true,
  "repo": "CentOS-8-Base-mirrors.aliyun.com",
  "state": "present"
}
```

 

在命令执行成功后，可以到主机清单中的任意机器上查看新建成功的软件仓库配置文件。

```bash
[root@play2 ~]# cat /etc/yum.repos.d/CentOS-8-Base-mirrors.aliyun.com.repo 
[CentOS-8-Base-mirrors.aliyun.com]
baseurl = https://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/
          http://mirrors.aliyuncs.com/centos/$releasever/BaseOS/$basearch/os/
          http://mirrors.cloud.aliyuncs.com/centos/$releasever/BaseOS/$basearch/os/
enabled = 1
gpgcheck = 1
gpgkey = https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
name = mirrors.163.com
```

 

 

## 运行剧本文件（playbook）

 

使用ansible临时命令仅执行单个命令或调用某一个模块，根本无法满足复杂工作的需求。所以Ansible服务允许用户根据需求，编写类似于shell脚本从上到下按顺序运行命令的playbook，由程序自动、重复的执行。



Ansible服务的剧本（playbook）文件采用**YAML**语言编写，具有**强制性的格式规范**，它通过空格将不同信息分组，因此有时会因一两个空格错位而导致报错。使用时要万分小心。YAML文件的开头需要先写3个减号（---），多个分组的信息需要间隔一致才能执行，而且上下也要对齐，后缀名一般为.yml。剧本文件在执行后，会在屏幕上输出运行界面，内容会根据工作的不同而变化。在运行界面中，绿色表示成功，黄色表示执行成功并进行了修改，而红色则表示执行失败。



剧本文件的结构由4部分组成，分别是target、variable、task、handler，其各自的作用如下。

- target：用于定义要执行剧本的主机范围。

- variable：用于定义剧本执行时要用到的变量。

- task：用于定义将在远程主机上执行的任务列表。

- handler：用于定义执行完成后需要调用的后续任务。




例如，创建一个名为packages.yml的剧本，让dev、test组的主机可以自动安装数据库软件，并且将dev组主机的软件更新至最新。

 

安装和更新软件需要使用yum模块。先看一下帮助信息中的示例吧：

 

```yaml
ansible-doc yum

> YUM  (/usr/lib/python3.6/site-packages/ansible/modules/packaging/os/yum.py)

​    Installs, upgrade, downgrades, removes, and lists packages and
​    groups with the `yum' package manager. This module only works
​    on Python 2. If you require Python 3 support see the [dnf]
​    module.

  * This module is maintained by The Ansible Core Team
  * note: This module has a corresponding action plugin.

EXAMPLES:

- name: install the latest version of Apache
  yum:
  name: httpd
  state: latest

- name: ensure a list of packages installed
  yum:
  name: "{{ packages }}"
  vars:
  packages:
  - httpd
  - httpd-tools
```

 

在配置Ansible剧本文件时，ansible-doc命令提供的帮助信息真是好用。在知道yum模块的使用方法和格式后，就可以开始编写剧本了。初次编写剧本文件时，请务必看准格式，模块及play（动作）格式也要上下对齐，否则会出现“参数一模一样，但不能执行”的情况。

 

```bash
vim packages.yml
```

```yaml
---
- name: 安装软件包
  hosts: devserver,webserver
  tasks:
          - name: sql
            yum:
                    name: mariadb
                    state: latest
```

 

**name**字段表示此项play（动作）的名字，用于在执行过程中提示用户执行到了哪一步，可以在name字段自行命名，没有任何限制。

**hosts**字段表示要在哪些主机上执行该剧本，多个主机组之间用逗号间隔；如果需要对全部主机进行操作，则使用all参数。

**tasks**字段用于定义要执行的任务，每个任务都要有一个独立的name字段进行命名，并且每个任务的name字段和模块名称都要严格上下对齐，参数要单独缩进。



在确认无误后就可以用`ansible-playbook`命令运行这个剧本文件了。

```bash
[root@play1 ~]# ansible-playbook packages.yml 
PLAY [安装软件包] ***********************************************************************

TASK [Gathering Facts] *************************************************************
ok: [192.168.88.139]
ok: [192.168.88.130]

TASK [sql] *************************************************************************
changed: [192.168.88.130]
changed: [192.168.88.139]

PLAY RECAP *************************************************************************
192.168.88.130       : ok=2  changed=1  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0  
192.168.88.139       : ok=2  changed=1  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0 
```

 

在执行成功后，我们主要观察最下方的输出信息。其中，ok和changed表示执行及修改成功。如遇到unreachable或failed大于0的情况，建议手动检查剧本是否在所有主机中都正确运行了，以及有无安装失败的情况。在正确执行过packages.yml文件后，随机切换到dev、test、prod组中的任意一台主机上，再次安装mariadb软件包，此时会提示该服务已经存在。这说明刚才的操作一切顺利！

 

```bash
[root@play2 ~]# dnf install mariadb
Last metadata expiration check: 1:12:15 ago on Wed 27 Oct 2021 06:28:24 PM CST.
Package mariadb-3:10.3.28-1.module_el8.3.0+757+d382997d.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

 

 

## 创建及使用角色（role）



>



在日常编写剧本时，会存在剧本越来越长的情况，这不利于进行阅读和维护，而且还无法让其他剧本灵活地调用其中的功能代码。角色（role）这一功能则是自Ansible 1.2版本开始引入的新特性，用于层次性、结构化地组织剧本。



角色功能分别把变量、文件、任务、模块及处理器配置放在各个独立的目录中，然后对其进行便捷加载。简单来说，**角色功能是把常用的一些功能“类模块化”**，然后在用的时候加载即可。



Ansible服务的角色功能类似于编程中的封装技术—将具体的功能封装起来，用户不仅可以方便地调用它，而且甚至可以不用完全理解其中的原理。这便是技术封装的好处。

 

角色的好处就在于将剧本组织成了一个简洁的、可重复调用的抽象对象，使得用户把注意力放到剧本的宏观大局上，统筹各个关键性任务，只有在需要时才去深入了解细节。

> Ansible有3种角色的获取方法，分别是加载系统内置角色、从外部环境获取角色以及自行创建角色。 

 

### 加载系统内置角色

在使用RHEL系统的内置角色时，我们不需要联网就能实现。用户只需要配置好软件仓库的配置文件，然后安装包含系统角色的软件包rhel-system-roles，随后便可以在系统中找到它们了，然后就能够使用剧本文件调用角色了。

 

安装包含系统角色的软件包

```bash
dnf install -y rhel-system-roles
```

 

安装完毕后，查看RHEL 8系统中有哪些自带的角色可用

```bash
ansible-galaxy list

# /usr/share/ansible/roles
- linux-system-roles.certificate, (unknown version)
- linux-system-roles.crypto_policies, (unknown version)
- linux-system-roles.ha_cluster, (unknown version)
- linux-system-roles.kdump, (unknown version)
- linux-system-roles.kernel_settings, (unknown version)
- linux-system-roles.logging, (unknown version)
……
```

千万不要低估这些由系统镜像自带的角色，它们在日常的工作中能派上大用场。

ansible系统角色描述

| 角色名称                   | 作用                  |
| -------------------------- | --------------------- |
| rhel-system-roles.kdump    | 配置kdump崩溃恢复服务 |
| rhel-system-roles.network  | 配置网络接口          |
| rhel-system-roles.selinux  | 配置SELinux策略及模式 |
| rhel-system-roles.timesync | 配置网络时间协议      |
| rhel-system-roles.postfix  | 配置邮件传输服务      |
| rhel-system-roles.firewall | 配置防火墙服务        |
| rhel-system-roles.tuned    | 配置系统调优选项      |

以rhel-system-roles.timesync角色为例，它用于设置系统的时间和NTP服务，让主机能够同步准确的时间信息。剧本模板文件存放在/usr/share/doc/rhel-system-roles/目录中，可以复制过来修改使用：



```bash
cp /usr/share/doc/rhel-system-roles/timesync/example-timesync-playbook.yml timesync.yml
```

NTP服务器主要用于同步计算机的时间，可以提供高精度的时间校准服务，帮助计算机校对系统时钟。在复制来的剧本模板文件中，删除掉多余的代码，将NTP服务器的地址填写到timesync_ntp_servers变量的hostname字段中即可。该变量的参数含义如表所示。稍后timesync角色就会自动为用户配置参数信息了。

 

timesync_ntp_servers变量参数含义

| 参数     | 作用            |
| -------- | --------------- |
| hostname | NTP服务器主机名 |
| iburst   | 启用快速同步    |

新建剧本文件，调用系统角色

```bash
vim timesync.yml
```

```yaml
---
- hosts: all
  vars:
    timesync_ntp_servers:
      - hostname: ntp.aliyun.com
        iburst: yes
  roles:
    - rhel-system-roles.timesync
```

执行剧本文件

```bash
ansible-playbook timesync.yml
```

 

### 从外部获取角色

Ansible Galaxy是Ansible的一个官方社区，用于共享角色和功能代码，用户可以在网站自由地共享和下载Ansible角色。该社区是管理和使用角色的不二之选。

在Ansible Galaxy官网中，左侧有3个功能选项，分别是首页（Home）、搜索（Search）以及社区（Community）。单击Search按钮进入到搜索界面，这里以nginx服务为例进行搜索，即可找到Nginx官方发布的角色信息



当单击nginx角色进入到详情页面后，会显示这个项目的软件版本、评分、下载次数等信息。在Installation字段可以看到相应的安装方式，在保持虚拟机能够连接外网的前提下，可以按这个页面提示的命令进行安装。

 

这时，如果需要使用这个角色，可以在虚拟机联网的状态下直接按照“ansible-galaxy install角色名称”的命令格式自动获取：

 

```bash
ansible-galaxy collection install nginxinc.nginx_core
```

 

执行完毕后，再次查看系统中已有的角色，便可找到nginx角色信息了：

 

```bash
ansible-galaxy list

# /etc/ansible/roles
- nginxinc.nginx, 0.19.1
# /usr/share/ansible/roles
- linux-system-roles.kdump, (unknown version)
- linux-system-roles.network, (unknown version)
```

 

这里还存在两种特殊情况。

在国内问Ansible Galaxy官网时可能存在不稳定的情况，导致访问不了或者网速较慢。

某位作者是将作品上传到了自己的网站，或者除Ansible Galaxy官网以外的其他平台。

在这两种情况下，就不能再用“ansible-galaxy install角色名称”的命令直接加载了，而是需要手动先编写一个YAML语言格式的文件，指明网址链接和角色名称，然后再用-r参数进行加载。



例如，在（www.linuxprobe.com）上传了一个名为nginx_core的角色软件包（一个用于对nginx网站进行保护的插件）。这时需要编写如下所示的一个yml配置文件：

```bash
vim nginx.yml
```

```yaml
---
- src: https://www.linuxprobe.com/Software/nginxinc-nginx_core-0.3.0.tar.gz
  name: nginx-core
```

 

随后使用ansible-galaxy命令的-r参数加载这个文件

```bash
ansible-galaxy install -r nginx.yml
```

 

即可查看到新角色信息了：

```bash
ansible-galaxy list

# /root/.ansible/roles
- nginx-core, (unknown version)
# /usr/share/ansible/roles
- linux-system-roles.certificate, (unknown version)
- linux-system-roles.crypto_policies, (unknown version)
- linux-system-roles.ha_cluster, (unknown version)
- linux-system-roles.kdump, (unknown version)
- linux-system-roles.kernel_settings, (unknown version)
- linux-system-roles.logging, (unknown version)
```

 

### 创建自定义角色

除了能够使用系统自带的角色和从Ansible Galaxy中获取的角色之外，也可以自行创建符合工作需求的角色。这种定制化的编写工作能够更好地贴合生产环境的实际情况，但难度也会稍高一些。

 

接下来将会创建一个名为apache的新角色，它能够帮助我们自动安装、运行httpd网站服务，设置防火墙的允许规则，以及根据每个主机生成独立的index.html首页文件。用户在调用这个角色后能享受到“一条龙”的网站部署服务。

 

在Ansible的主配置文件中，第68行定义的是角色保存路径。如果用户新建的角色信息不在规定的目录内，则无法使用ansible-galaxy list命令找到。因此需要手动填写新角色的目录路径，或是进入/etc/ansible/roles目录内再进行创建。为了避免后期角色信息过于分散导致不好管理，我们还是决定在默认目录下进行创建，不再修改。

 

```bash
vim /etc/ansible/ansible.cfg
```

```bash
 67 # additional paths to search for roles in, colon separated
 68 # roles_path  = /etc/ansible/roles
```

 

 

在ansible-galaxy命令后面跟一个init参数，创建一个新的角色信息，且建立成功后便会在当前目录下生成出一个新的目录：

```bash
cd /etc/ansible/roles/
```

```bash
ansible-galaxy init apache
- Role apache was created successfully
```



此时的apache即是角色名称，也是用于存在角色信息的目录名称。切换到该目录下，查看它的结构：

```bash
ls apache/
defaults files handlers meta README.md tasks templates tests vars
```

 

在创建新角色时，最关键的便是能够正确理解目录结构。通俗来说，就是要把正确的信息放入正确的目录中，这样在调用角色时才能有正确的效果。角色信息对应的目录结构及含义如表所示

Ansible角色目录结构及含义

| 目录      | 含义                                           |
| --------- | ---------------------------------------------- |
| defaults  | 包含角色变量的默认值（优先级低）。             |
| files     | 包含角色执行tasks任务时做引用的静态文件。      |
| handlers  | 包含角色的处理程序定义。                       |
| meta      | 包含角色的作者、许可证、频台和依赖关系等信息。 |
| tasks     | 包含角色所执行的任务。                         |
| templates | 包含角色任务所使用的Jinja2模板。               |
| tests     | 包含用于测试角色的剧本文件。                   |
| vars      | 包含角色变量的默认值（优先级高）。             |

 

下面准备创建自定义角色。

 

**第1步**：打开用于定义角色任务的tasks/main.yml文件。在该文件中不需要定义要执行的主机组列表，因为后面会单独编写剧本进行调用，此时应先对apache角色能做的事情（任务）有一个**明确的思路**，在调用角色后yml文件会按照从上到下的顺序自动执行。

 

- **任务1**：安装httpd网站服务。

- **任务2**：运行httpd网站服务，并加入到开机启动项中。
- **任务3**：配置防火墙，使其放行HTTP协议。

- **任务4**：根据每台主机的变量值，生成不同的主页文件。

先写出第一个任务。使用yum模块安装httpd网站服务程序（注意格式）：

```yaml
vim /etc/ansible/roles/apache/tasks/main.yml
  
---
# tasks file for apache
- name: install
  yum:
          name: httpd
          state: latest
```

  

**第2步**：使用service模块启动httpd网站服务程序，并加入到启动项中，保证能够一直为用户提供服务。在初次使用模块前，先用ansible-doc命令查看一下帮助和实例信息。对这里的信息进行了删减，仅保留了有用的内容。

 

 

```yaml
ansible-doc service

> SERVICE  (/usr/lib/python3.6/site-packages/ansible/modules/system/service.py)
    Controls services on remote hosts. Supported init systems
    include BSD init, OpenRC, SysV, Solaris SMF, systemd, upstart.
    For Windows targets, use the [win_service] module instead.

 * This module is maintained by The Ansible Core Team
 * note: This module has a corresponding action plugin.
 
XAMPLES:

- name: Start service httpd, if not started
  service:
  name: httpd
  state: started

- name: Stop service httpd, if started
  service:
  name: httpd
  state: stopped

- name: Restart service httpd, in all cases
  service:
  name: httpd
  state: restarted
  
- name: Reload service httpd, in all cases
  service:
  name: httpd
  state: reloaded
  
- name: Enable service httpd, and not touch the state
  service:
  name: httpd
  enabled: yes
```

 

默认的EXAMPLES示例使用的就是httpd网站服务。通过输出信息可得知，启动服务为“state: started”参数，而加入到开机启动项则是“enabled: yes”参数。继续编写：

```yaml
vim /etc/ansible/roles/apache/tasks/main.yml

---
# tasks file for apache
- name: install
  yum:
          name: httpd
          state: latest
- name: start
  service:
          name: httpd
          state: started
          enabled: yes
```

**第3步**：配置防火墙的允许策略，让其他主机可以正常访问。在配置防火墙时，需要使用firewalld模块。同样也是先看一下帮助示例：

```yaml
ansible-doc firewalld
> FIREWALLD    (/usr/lib/python3.6/site-packages/ansible/modules/system/firewalld.py)

        This module allows for addition or deletion of services and ports (either TCP or UDP) in either running or
        permanent firewalld rules.

  * This module is maintained by The Ansible Community
  
EXAMPLES:

- firewalld:
    service: https
    permanent: yes
    state: enabled

- firewalld:
    port: 8081/tcp
    permanent: yes
    state: disabled

- firewalld:
    port: 161-162/udp
    permanent: yes
    state: enabled

- firewalld:
    zone: dmz
    service: http
    permanent: yes
    state: enabled
    
- firewalld:
    rich_rule: rule service name="ftp" audit limit value="1/m" accept
    permanent: yes
    state: enabled
```



依据输出信息可得知，在firewalld模块设置防火墙策略时，指定协议名称为“service: http”参数，放行该协议为“state: enabled”参数，设置为永久生效为“permanent: yes”参数，当前立即生效为“immediate: yes”参数。参数虽然多了一些，但是基本与在之前学习的一致，并不需要担心。继续编写：

```yaml
vim /etc/ansible/roles/apache/tasks/main.yml

---
# tasks file for apache
- name: install
  yum:
          name: httpd
          state: latest
- name: start
  service:
          name: httpd
          state: started
          enabled: yes
- name: firewall
  firewalld:
          service: http
          permanent: yes
          state: enabled
          immediate: yes
```





**第4步**：让每台主机显示的主页文件均不相同。在使用Ansible的常规模块时，都是采用“查询版主示例并模仿”的方式搞定的，这里为了增加难度，我们再提出个新需求，即能否让每台主机上运行的httpd网站服务都能显示不同的内容呢？例如显示当前服务器的主机名及IP地址。这就要用到template模块及Jinja2技术了。

 

我们依然使用ansible-doc命令来查询template模块的使用方法。示例部分依然大有帮助：

```yaml
ansible-doc template

> TEMPLATE  (/usr/lib/python3.6/site-packages/ansible/modules/files/template.py)
     Templates are processed by the L(Jinja2 templating
     language,http://jinja.pocoo.org/docs/). Documentation on the
     template formatting can be found in the L(Template Designer
     Documentation,http://jinja.pocoo.org/docs/templates/).
     Additional variables listed below can be used in templates.
     `ansible_managed' (configurable via the `defaults' section of
     `ansible.cfg') contains a string which can be used to describe
     the template name, host, modification time of the template
     file and the owner uid. `template_host' contains the node name
     of the template's machine. `template_uid' is the numeric user
     id of the owner. `template_path' is the path of the template.
     `template_fullpath' is the absolute path of the template.
     `template_destpath' is the path of the template on the remote
     system (added in 2.8). `template_run_date' is the date that
     the template was rendered.
     
  * This module is maintained by The Ansible Core Team
  * note: This module has a corresponding action plugin.

EXAMPLES:

- name: Template a file to /etc/files.conf
  template:
  src: /mytemplates/foo.j2
  dest: /etc/file.conf
  owner: bin
  group: wheel
  mode: '0644'

- name: Template a file, using symbolic modes (equivalent to 0644)
  template:
  src: /mytemplates/foo.j2
  dest: /etc/file.conf
  owner: bin
  group: wheel
  mode: u=rw,g=r,o=r

- name: Copy a version of named.conf that is dependent on the OS. setype obtained >
  template:
  src: named.conf_{{ ansible_os_family }}.j2
  dest: /etc/named.conf
  group: named
  setype: named_conf_t
  mode: 0640
```

 

从template模块的输出信息中可得知，这是一个用于复制文件模板的模块，能够把文件从Ansible服务器复制到受管主机上。其中，src参数用于定义本地文件的路径，dest参数用于定义复制到受管主机的文件路径，而owner、group、mode参数可选择性地设置文件归属及权限信息。

 

正常来说，我们可以直接复制文件的操作，受管主机上会获取到一个与Ansible服务器上的文件一模一样的文件。但有时候，我们想让每台客户端根据自身系统的情况产生不同的文件信息，这就需要用到Jinja2技术了，Jinja2格式的模板文件后缀是.j2。继续编写：

 

```yaml
vim /etc/ansible/roles/apache/tasks/main.yml

---
# tasks file for apache
- name: install
  yum:
          name: httpd
          state: latest
- name: start
  service:
          name: httpd
          state: started
          enabled: yes
- name: firewall
  firewalld:
          service: http
          permanent: yes
          state: enabled
          immediate: yes
- name: index
  template:
          src: index.html.j2
          dest: /var/www/html/index.html
```

 

 

Jinja2是Python语言中一个被广泛使用的模板引擎，最初的设计思想源自Django的模块引擎。Jinja2基于此发展了其语法和一系列强大的功能，能够让受管主机根据自身变量产生出不同的文件内容。换句话说，正常情况下的复制操作会让新旧文件一模一样，但在使用Jinja2技术时，不是在原始文件中直接写入文件内容，而是写入一系列的变量名称。在使用template模块进行复制的过程中，由Ansible服务负责在受管主机上收集这些变量名称所对应的值，然后再逐一填写到目标文件中，从而让每台主机的文件都根据自身系统的情况独立生成。

 

例如，想要让每个网站的输出信息值为“Welcome to 主机名 on 主机地址”，也就是用每个主机自己独有的名称和IP地址来替换文本中的内容，这样就有趣太多了。这个实验的难点在于查询到对应的变量名称、主机名及地址所对应的值保存在哪里？可以用setup模块进行查询。

 

```bash
ansible-doc setup

> SETUP  (/usr/lib/python3.6/site-packages/ansible/modules/system/setup.py)

     This module is automatically called by playbooks to gather
     useful variables about remote hosts that can be used in
     playbooks. It can also be executed directly by
     `/usr/bin/ansible' to check what variables are available to a
     host. Ansible provides many `facts' about the system,
     automatically. This module is also supported for Windows
     targets.

 * This module is maintained by The Ansible Core Team
```

 

setup模块的作用是自动收集受管主机上的**变量**信息，使用-a参数外加filter命令可以对收集来的信息进行二次过滤。相应的语法格式为ansible all -m setup -a 'filter="*关键词*"'，其中*号是Shell中的通配符，用于进行关键词查询。例如，如果想搜索各个主机的名称，可以使用通配符搜索所有包含fqdn关键词的变量值信息。

 

**FQDN（Fully Qualified Domain Name，完全限定域名）**用于在逻辑上准确表示出主机的位置。FQDN常常被作为主机名的完全表达形式，比 /etc/hostname 文件中定义的主机名更加严谨和准确。通过输出信息可得知，ansible_fqdn变量保存有主机名称。随后进行下一步操作：

 

```bash
[root@play1 ~]# ansible all -m setup -a 'filter="*fqdn"'

192.168.88.139 | SUCCESS => {
  "ansible_facts": {
     "ansible_fqdn": "play3",
     "discovered_interpreter_python": "/usr/libexec/platform-python"
  },
  "changed": false
}
192.168.88.130 | SUCCESS => {
  "ansible_facts": {
     "ansible_fqdn": "play2",
     "discovered_interpreter_python": "/usr/libexec/platform-python"
  },
  "changed": false
}
```

 

 

用于指定主机地址的变量可以用ip作为关键词进行检索。可以看到，ansible_all_ipv4_addresses变量中的值是我们想要的信息。如果想输出IPv6形式的地址，则可用ansible_all_ipv6_addresses变量。

 

```bash
[root@play1 ~]# ansible all -m setup -a 'filter="*ip*"'

192.168.88.139 | SUCCESS => {

  "ansible_facts": {

     "ansible_all_ipv4_addresses": [
       "192.168.88.139"
     ],
     "ansible_all_ipv6_addresses": [],
     "ansible_default_ipv4": {
       "address": "192.168.88.139",
       "alias": "ens33",
       "broadcast": "192.168.88.255",
       "gateway": "192.168.88.2",
       "interface": "ens33",
       "macaddress": "00:0c:29:84:d1:6f",
       "mtu": 1500,
       "netmask": "255.255.255.0",
       "network": "192.168.88.0",
       "type": "ether"
     },
     "ansible_default_ipv6": {},
     "ansible_fips": false,
     "discovered_interpreter_python": "/usr/libexec/platform-python"
  },
  "changed": false
}

192.168.88.130 | SUCCESS => {
  "ansible_facts": {
     "ansible_all_ipv4_addresses": [
       "192.168.88.130",
       "192.168.122.1"
     ],
     "ansible_all_ipv6_addresses": [
       "fe80::20c:29ff:fe7c:18d7"
     ],
     "ansible_default_ipv4": {
       "address": "192.168.88.130",
       "alias": "ens33",
       "broadcast": "192.168.88.255",
       "gateway": "192.168.88.2",
       "interface": "ens33",
       "macaddress": "00:0c:29:7c:18:d7",
       "mtu": 1500,
       "netmask": "255.255.255.0",
       "network": "192.168.88.0",
       "type": "ether"
     },
     "ansible_default_ipv6": {},
     "ansible_fips": false,
     "discovered_interpreter_python": "/usr/libexec/platform-python"
  },
  "changed": false
}
```

 

在确认了主机名与IP地址所对应的具体变量名称后，在角色所对应的templates目录内新建一个与上面的template模块参数相同的文件名称（index.html.j2）。Jinja2在调用变量值时，格式为在变量名称的两侧格加两个大括号：

 

```bash
cd /etc/ansible/roles/apache/
```

```bash
vim templates/index.html.j2 
Welcome to {{ ansible_fqdn }} on {{ ansible_all_ipv4_addresses }}
```



进行到这里，任务基本就算完成了。最后要做的就是编写一个用于调用apache角色的yml文件，以及执行这个文件。

 

```bash
cd ~/
```

```bash
vim roles.yml
```
```yaml
---
- name: 调用自建角色
  hosts: all
  roles:
          - apache 
```

 

```bash
[root@play1 ~]# ansible-playbook roles.yml 

PLAY [调用自建角色] ****************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [192.168.88.139]
ok: [192.168.88.130]

TASK [apache : install] ******************************************************************************************************************************
ok: [192.168.88.139]
ok: [192.168.88.130]

TASK [apache : start] ********************************************************************************************************************************
ok: [192.168.88.139]
ok: [192.168.88.130]

TASK [apache : firewall] *****************************************************************************************************************************
ok: [192.168.88.139]
ok: [192.168.88.130]

TASK [apache : index] ********************************************************************************************************************************
changed: [192.168.88.139]
changed: [192.168.88.130]

 
PLAY RECAP *******************************************************************************************************************************************
192.168.88.130       : ok=5  changed=1  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0  
192.168.88.139       : ok=5  changed=1  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0  
```

 

执行完毕后，在浏览器中随机输入几台主机的IP地址，即可访问到包含主机FQDN和IP地址的网页了

 

 

## 创建和使用逻辑卷

 

创建一个能批量、自动管理逻辑卷设备的剧本，不但能大大提高硬盘设备的管理效率，而且还能避免手动创建带来的错误。例如，我们想在每台受管主机上都创建出一个名为data的逻辑卷设备，大小为150MB，归属于new卷组。如果创建成功，则进一步用Ext4文件系统进行格式化操作；如果创建失败，则给用户输出一条报错提醒，以便排查原因。

 

在这种情况下，使用Ansible剧本要比使用Shell脚本的优势大，原因主要有下面两点。

- Ansible模块化的功能让操作更标准，只要在执行过程中无报错，那么便会依据远程主机的系统版本及配置自动做出判断和操作，不用担心因系统变化而导致命令失效的问题。


- Ansible服务在执行剧本文件时会进行判断：如果该文件或该设备已经被创建过，或是某个动作（play）已经被执行过，则绝对不会再重复执行；而使用Shell脚本有可能导致设备被重复格式化，导致数据丢失。




通过之前学习过的逻辑卷的知识，我们应该让剧本文件依次创建物理卷（PV）、卷组（VG）及逻辑卷（LV）。需要先使用lvg模块让设备支持逻辑卷技术，然后创建一个名为new的卷组。lvg模块的帮助信息如下：

 

```yaml
ansible-doc lvg

> LVG  (/usr/lib/python3.6/site-packages/ansible/modules/system/lvg.py)
    * This module creates, removes or resizes volume groups.
    * This module is maintained by The Ansible Community

EXAMPLES:

- name: Create a volume group on top of /dev/sda1 with physical extent size = 32MB
  lvg:
  vg: vg.services
  pvs: /dev/sda1
  pesize: 32

- name: Create a volume group on top of /dev/sdb with physical extent size = 128KiB
  lvg:
  vg: vg.services
  pvs: /dev/sdb
  pesize: 128K
  
# If, for example, we already have VG vg.services on top of /dev/sdb1,
# this VG will be extended by /dev/sdc5. Or if vg.services was created on
# top of /dev/sda5, we first extend it with /dev/sdb1 and /dev/sdc5,
# and then reduce by /dev/sda5.

- name: Create or resize a volume group on top of /dev/sdb1 and /dev/sdc5.
  lvg:
  vg: vg.services
  pvs: /dev/sdb1,/dev/sdc5

- name: Remove a volume group with name vg.services
  lvg:
  vg: vg.services
  state: absent
```

 

通过输出信息可得知，创建PV和VG的lvg模块总共有3个必备参数。其中，vg参数用于定义卷组的名称，pvs参数用于指定硬盘设备的名称，pesize参数用于确定最终卷组的容量大小（可以用PE个数或容量值进行指定）。这样一来，我们先创建出一个由/dev/sdb设备组成的名称为research、大小为150MB的卷组设备。

 ```yaml
 vim lv.yml
 ---
 - name: 创建和使用逻辑卷
   hosts: dev
   tasks:
         - name: create pv vg
           lvg:
           vg: new
           pvs: /dev/sdb
           pesize: 150M
 ```



接下来使用lvol模块创建出逻辑卷设备。还是按照惯例，先查看模块的帮助信息：

 

```yaml
ansible-doc lvol

> LVOL  (/usr/lib/python3.6/site-packages/ansible/modules/system/lvol.py)
 * This module creates, removes or resizes logical volumes.
 * This module is maintained by The Ansible Community
 
EXAMPLES:

- name: Create a logical volume of 512m
  lvol:
  vg: firefly
  lv: test
  size: 512

- name: Create a logical volume of 512m with disks /dev/sda and /dev/sdb
  lvol:
  vg: firefly
  lv: test
  size: 512
  pvs: /dev/sda,/dev/sdb

- name: Create cache pool logical volume
  lvol:
  vg: firefly
  lv: lvcache
  size: 512m
  opts: --type cache-pool

- name: Create a logical volume of 512g.
  lvol:
  vg: firefly
  lv: test
  size: 512g
```

 

通过输出信息可得知，lvol是用于创建逻辑卷设备的模块。其中，vg参数用于指定卷组名称，lv参数用于指定逻辑卷名称，size参数则用于指定最终逻辑卷设备的容量大小（不用加单位，默认为MB）。填写好参数，创建出一个大小为150MB、归属于new卷组且名称为data的逻辑卷设备

```yaml
vim lv.yml
---
- name: 创建和使用逻辑卷
  hosts: dev
  tasks:
         - name: create pv vg
           lvg:
           vg: new
           pvs: /dev/sdb
           pesize: 150M
           
         - name: create lv
           lvol:
           vg: new
           lv: data
           size: 150M
```

 

这样还不够，如果还能将创建出的/dev/new/data逻辑卷设备自动用Ext4文件系统进行格式化操作，则又能帮助运维管理员减少一些工作量。可使用filesystem模块来完成设备的文件系统格式化操作。该模块的帮助信息如下：



```yaml
ansible-doc filesystem

> FILESYSTEM  (/usr/lib/python3.6/site-packages/ansible/modules/system/filesyste>
 * This module creates a filesystem.
 * This module is maintained by The Ansible Community

EXAMPLES:

- name: Create a ext2 filesystem on /dev/sdb1
  filesystem:
  fstype: ext2
  dev: /dev/sdb1

- name: Create a ext4 filesystem on /dev/sdb1 and check disk blocks
  filesystem:
  fstype: ext4
  dev: /dev/sdb1
  opts: -cc
```

 

filesystem模块的参数真是简练，fstype参数用于指定文件系统的格式化类型，dev参数用于指定要格式化的设备文件路径。继续编写

```yaml
vim lv.yml
---
- name: 创建和使用逻辑卷
  hosts: dev
  tasks:
         - name: create pv vg
           lvg:
           vg: new
           pvs: /dev/sdb
           pesize: 150M
           
         - name: create lv
           lvol:
           vg: new
           lv: data
           size: 150M 
           
         - name: format lv
           filesystem:
           fstype: xfs
           dev: /dev/new/data
```



这样按照顺序执行下来，逻辑卷设备就能够自动创建好了。等一下，还有个问题没有解决。现在只有dev组的主机上添加了新的硬盘设备文件，其余主机是无法按照既定模块顺利完成操作的。这时就要使用类似于Shell的if条件语句的方式进行判断—如果失败……，则……。

首先用block操作符将上述的3个模块命令作为一个整体（相当于对这3个模块的执行结果作为一个整体进行判断），然后使用rescue操作符进行救援，且只有block块中的模块执行失败后才会调用rescue中的救援模块。其中，debug模块的msg参数的作用是，如果block中的模块执行失败，则输出一条信息到屏幕，用于提醒用户。完成编写后的剧本是下面这个样子：

 

**(注意缩进)**

```yaml
vim lv.yml
---
- name: 创建和使用逻辑卷
  hosts: all
  tasks:
          - block:
                  - name: create pv vg
                    lvg:
                            vg: new
                            pvs: /dev/sdb
                            pesize: 150M
                  - name: create lv
                    lvol:
                            vg: new
                            lv: data
                            size: 150M
                  - name: format lv
                    filesystem:
                            fstype: xfs
                            dev: /dev/new/data
            rescue:
                    - debug:
                            msg: "Could not create logical volume of that size(无法创建那样大小的逻辑卷)"
```

 

YAML语言对格式有着硬性的要求，既然rescue是对block内的模块进行救援的功能代码，因此recue和block两个操作符必须严格对齐，错开一个空格都会导致剧本执行失败。确认无误后，执行lv.yml剧本文件检阅一下效果：

 

```bash
[root@play1 ~]# ansible-playbook lv.yml 

PLAY [创建和使用逻辑卷] ***********************************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************************
ok: [192.168.88.139]
ok: [192.168.88.130]

TASK [create pv vg] *******************************************************************************************************************************************************
changed: [192.168.88.130]
changed: [192.168.88.139]

TASK [create lv] **********************************************************************************************************************************************************
changed: [192.168.88.139]
changed: [192.168.88.130]

TASK [format lv] **********************************************************************************************************************************************************
changed: [192.168.88.139]
changed: [192.168.88.130]

PLAY RECAP ****************************************************************************************************************************************************************
192.168.88.130       : ok=4  changed=3  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0 
192.168.88.139       : ok=4  changed=3  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0 
```

 

在剧本运行完毕后的执行记录（PLAY RECAP）中可以很清晰地看到只有192.168.10.22及192.168.10.23这两台dev组中的主机执行成功了，其余主机均触发了rescue功能。登录到任意一台dev组的主机上，找到新建的逻辑卷设备信息：

 

```bash
[root@localhost ~]# lvdisplay 
 --- Logical volume ---
 LV Path        /dev/new/data
 LV Name        data
 VG Name        new
 LV UUID        oHVRhH-K017-riVa-VRvp-DEuY-FY8t-jNIng4
 LV Write Access    read/write
 LV Creation host, time localhost.localdomain, 2021-10-29 00:34:22 +0800
 LV Status       available
 # open         0
 LV Size        150.00 MiB
 Current LE       1
 Segments        1
 Allocation       inherit
 Read ahead sectors   auto
 \- currently set to   8192
 Block device      253:2
```

 

## 判断主机组名

 

在上面的剧本实验中，我们可以让不同的主机根据自身不同的变量信息而生成出独特的网站主页文件，但却无法对某个主机组进行针对性的操作。其实，在每个客户端中都会有一个名为inventory_hostname的变量，用于定义每台主机所对应的Ansible服务的主机组名称，也就是/etc/ansible/hosts文件中所对应的分组信息，例如dev、test、prod、balancers。

inventory_hostname是Ansible服务中的魔法变量，这意味着无法使用setup模块直接进行查询，诸如ansible all -m setup -a 'filter="*关键词*"'这样的命令将对它失效。魔法变量需要在执行剧本文件时的Gathering Facts阶段进行搜集，直接查询是看不到的，只能在剧本文件中进行调用。

 

在获得了存储主机组名称的变量名称后，接下来开始实战。这里的需求如下：

- 若主机在dev分组中，则修改/etc/issue文件内容为Development；

- 若主机在test分组中，则修改/etc/issue文件内容为Test；




根据之前所提及的Ansible常用模块名称及作用，可以看到**copy模块的主要作用是新建、修改及复制文件**，更符合当前的需要，此时便派上了用场。先查询copy模块的帮助信息：

 

```yaml
ansible-doc copy

> COPY  (/usr/lib/python3.6/site-packages/ansible/modules/files/copy.py)

​    The `copy' module copies a file from the local or remote
​    machine to a location on the remote machine. Use the [fetch]
​    module to copy files from remote locations to the local box.
​    If you need variable interpolation in copied files, use the
​    [template] module. Using a variable in the `content' field
​    will result in unpredictable output. For Windows targets, use
​    the [win_copy] module instead.

 * This module is maintained by The Ansible Core Team
 * note: This module has a corresponding action plugin.

EXAMPLES:

- name: Copy file with owner and permissions
  copy:
  src: /srv/myfiles/foo.conf
  dest: /etc/foo.conf
  owner: foo
  group: foo
  mode: '0644'

- name: Copy file with owner and permission, using symbolic representation
  copy:
  src: /srv/myfiles/foo.conf
  dest: /etc/foo.conf
  owner: foo
  group: foo
  mode: u=rw,g=r,o=r

- name: Another symbolic mode example, adding some permissions and removing others
  copy:
  src: /srv/myfiles/foo.conf
  dest: /etc/foo.conf
  owner: foo
  group: foo
  mode: u+rw,g-wx,o-rwx

- name: Copy a new "ntp.conf file into place, backing up the original if it differ>
  copy:
  src: /mine/ntp.conf
  dest: /etc/ntp.conf
  owner: root
  group: root
  mode: '0644'
  backup: yes
```

 

在输出信息中列举了两种管理文件内容的示例。第一种用于文件的复制行为，第二种是通过content参数定义内容，通过dest参数指定新建文件的名称。显然，第二种更加符合当前的实验场景。编写剧本文件如下：

 

```bash
vim issue.yml
```

```yaml
---
- name: 修改文件内容
  hosts: all
  tasks:
          - name: 定义内容dev
            copy:
                    content: 'Development'
                    dest: /etc/issue
          - name: 定义内容web
            copy:
                    content: 'Web'
                    dest: /etc/issue
```



但是，如果按照这种顺序执行下去，每一台主机的/etc/issue文件都会被重复修改2次，最终定格在“Web”字样，这显然缺少了一些东西。我们应该依据inventory_hostname变量中的值进行判断。若主机为dev组，则执行第一个动作；若主机为web组，则执行第二个动作；因此，要进行2次判断。



**when是用于判断的语法**，我们将其用在每个动作的下方进行判断，使得只有在满足条件才会执行：



```bash
vim issue.yml
```

```yaml
---
- name: 修改文件内容
  hosts: all
  tasks:
          - name: 定义内容dev
            copy:
                    content: 'Development'
                    dest: /etc/issue
            when: "inventory_hostname in groups.devserver"
          - name: 定义内容web
            copy:
                    content: 'Web'
                    dest: /etc/issue
            when: "inventory_hostname in groups.webserver"
```

 

执行剧本文件，在过程中可清晰地看到由于when语法的作用，未在指定主机组中的主机将被跳过（skipping）：

 

```bash
[root@play1 ~]# ansible-playbook issue.yml 

PLAY [修改文件内容] **********************************************************************

TASK [Gathering Facts] *************************************************************
ok: [192.168.88.139]
ok: [192.168.88.130]

TASK [定义内容dev] *********************************************************************
skipping: [192.168.88.139]
changed: [192.168.88.130]

TASK [定义内容web] *********************************************************************
skipping: [192.168.88.130]
changed: [192.168.88.139]


PLAY RECAP *************************************************************************
192.168.88.130       : ok=2  changed=1  unreachable=0  failed=0  skipped=1  rescued=0  ignored=0  
192.168.88.139       : ok=2  changed=1  unreachable=0  failed=0  skipped=1  rescued=0  ignored=0 
```

 

 

登录到dev组的192.168.88.139主机上，查看文件内容：

```bash
cat /etc/issue

Development
```

 

登录到dev组的192.168.88.130主机上，查看文件内容：

```bash
cat /etc/issue

Web
```

 

 

## 管理文件属性

 

我们学习剧本的目的是为了满足日常的工作需求，把重复的事情写入到脚本中，然后再批量执行下去，从而提高运维工作的效率。其中，创建文件、管理权限以及设置快捷方式几乎是每天都用到的技能。尤其是在学习文件的一般权限、特殊权限、隐藏权限时，往往还会因命令的格式问题而导致出错。这么多命令该怎么记呢？

 

**Ansible服务将常用的文件管理功能都合并到了file模块中**，大家不用再为了寻找模块而“东奔西跑”了。先来看一下file模块的帮助信息：

 

 

```yaml
ansible-doc file

> FILE  (/usr/lib/python3.6/site-packages/ansible/modules/files/file.py)

​    Set attributes of files, symlinks or directories.
​    Alternatively, remove files, symlinks or directories. Many
​    other modules support the same options as the `file' module -
​    including [copy], [template], and [assemble]. For Windows
​    targets, use the [win_file] module instead.

  * This module is maintained by The Ansible Core Team

EXAMPLES:

- name: Change file ownership, group and permissions
  file:
  path: /etc/foo.conf
  owner: foo
  group: foo
  mode: '0644'

- name: Give insecure permissions to an existing file
  file:
  path: /work
  owner: root
  group: root
  mode: '1777'

- name: Create a symbolic link
  file:
  src: /file/to/link/to
  dest: /path/to/symlink
  owner: foo
  group: foo
  state: link
```

 

 

通过上面的输出示例，大家已经能够了解file模块的基本参数了。其中，path参数定义了文件的路径，owner参数定义了文件所有者，group参数定义了文件所属组，mode参数定义了文件权限，src参数定义了源文件的路径，dest参数定义了目标文件的路径，state参数则定义了文件类型。

 

可见，file模块基本上把之前学习过的管理文件权限的功能都包含在内了。

 

- 请创建出一个名为/ftproot的新目录，所有者及所属组均为root管理员身份；
- 设置所有者和所属于组拥有对文件的完全控制权，而其他人则只有阅读和执行权限；
- 给予SGID特殊权限；
- 仅在dev主机组的主机上实施。
- 创建一个名称为 /ftp 的快捷方式文件，指向刚刚建立的 /ftproot 目录

 

```bash
vim chmod.yml
```

```yaml
---
- name: 管理文件属性
  hosts: devserver
  tasks:
          - name: chmod
            file:
                    path: /ftproot
                    state: directory
                    owner: root
                    group: root
                    mode: '2775'
          - name: ln
            file:
                    src: /ftproot
                    dest: /ftp
                    state: link
```



进入到dev组的主机中，可以看到/ftproot目录及/ftp的快捷方式均已经被顺利创建：

 

```bash
ll -d /ftp
lrwxrwxrwx 1 root root 8 Oct 29 01:10 /ftp -> /ftproot

ll -d /ftproot/
drwxrwsr-x 2 root root 6 Oct 29 01:10 /ftproot/
```

 

## 管理密码库文件

自Ansible 1.5版本发布后，**vault**作为一项新功能进入到了运维人员的视野。它不仅能对密码、剧本等敏感信息进行加密，而且还可以加密变量名称和变量值，从而确保数据不会被他人轻易阅读。使用ansible-vault命令可以实现内容的新建（create）、加密（encrypt）、解密（decrypt）、修改密码（rekey）及查看（view）等功能。

 

下面通过示例来学习vault的具体用法。

 

**第1步**：创建出一个名为locker.yml的配置文件，其中保存了两个变量值：

 

```bash
vim locker.yml
```

```yaml
---

pw_developer: Imadev
pw_manager: Imamgr
```



**第2步**：使用ansible-vault命令对文件进行加密。由于需要每次输入密码比较麻烦，因此还应新建一个用于保存密码值的文本文件，以便让ansible-vault命令自动调用。为了保证数据的安全性，在新建密码文件后将该文件的权限设置为600，确保仅管理员可读可写：

```bash
vim secret.txt

whenyouwishuponastar
```

```bash
chmod 600 secret.txt
```

在Ansible服务的主配置文件中，在第140行的vault_password_file参数后指定密码值保存的文件路径，准备进行调用（如果把密码文件路径加入主配置文件，则查看加密文件时无需输入密码）：

```bash
vim /etc/ansible/ansible.cfg

140 vault_password_file = /root/secret.txt
```

 

**第3步**：在设置好密码文件的路径后，Ansible服务便会自动进行加载。用户也就不用在每次加密或解密时都重复输入密码了。例如，在加密刚刚创建的locker.yml文件时，只需要使用encrypt参数即可：

```bash
ansible-vault encrypt locker.yml 

Encryption successful
```

 

文件将使用AES 256加密方式进行加密，也就是意味着密钥有2256种可能。查看到加密后的内容为：

```bash
cat locker.yml 

$ANSIBLE_VAULT;1.1;AES256
66363866396439333330303937326263646231386131333233346237366232333631636636653730
3262633761366532353165366465643132343861386464330a326265386265656630393263613333
65343065643332653637363730626365663830636462306239653537376231326539386666383631
3461396566633734620a643639663465616664663938326464393333336562343631373666663431
35373062363831306562653630373066613462343335303235646165623738306331663637386534
6330346432386439333966333261383165376235393534613339
```

 

如果不想使用原始密码了呢？也可以使用rekey参数手动对文件进行改密操作，同时应结合--ask-vault-pass参数进行修改，否则Ansible服务会因接收不到用户输入的旧密码值而拒绝新的密码变更请求：

```bash
ansible-vault rekey --ask-vault-pass locker.yml 
Vault password: 
New Vault password: 
Confirm New Vault password: 
Rekey successful
```

 

**第4步**：如果想查看和修改加密文件中的内容，该怎么操作呢？对于已经加密过的文件，需要使用ansible-vault命令的edit参数进行修改，随后用view参数即可查看到修改后的内容。ansible-vault命令对加密文件的编辑操作默认使用的是Vim编辑器，在修改完毕后请记得执行wq操作保存后退出：

```bash
ansible-vault edit locker.yml 

---
pw_developer: Imadev
pw_manager: Imamgr
pw_production: Imaprod
```

最后，再用view参数进行查看，便是最新的内容了：

```bash
ansible-vault view locker.yml 

---
pw_developer: Imadev
pw_manager: Imamgr
pw_production: Imaprod
```







 

[^1]: Epel-release：RHEL 8系统的镜像文件默认不带有某些服务程序，需要从Extra Packages for Enterprise Linux（EPEL）扩展软件包仓库获取。EPEL软件包仓库由红帽公司提供，是一个用于创建、维护和管理企业版Linux的高质量软件扩展仓库，通用于RHEL、CentOS、Oracle Linux等多种红帽系企业版系统，目的是对于默认系统仓库软件包进行扩展。

 

[^2]: Ansible服务的主配置文件存在优先级的顺序关系，默认存放在/etc/ansible目录中的主配置文件优先级最低。如果在当前目录或用户家目录中也存放着一份主配置文件，则以当前目录或用户家目录中的主配置文件为主。同时存在多个Ansible服务主配置文件时，具体优先级顺序如表所示。



---

References & Resources：

[www.linuxprobe.com：使用Ansible服务实现自动化运维](https://www.linuxprobe.com/basic-learning-16.html)
