---
title: "Vsftpd"
date: 2021-10-14T15:16:25+08:00
draft: true
---

## Vsftpd

### 简介

为了解决在复杂多样的设备之间传输文件，文件传输协议（FTP）诞生了。由于FTP协议是明文传输，不够可靠。所以为了满足传输的安全性，出现了**vsftpd（very secure ftp daemon，非常安全的FTP守护进程）**  



FTP默认使用**20**、**21**端口

- 20端口用于数据传输
- 21端口用于FTP的连接和控制（接受客户端发出的FTP命令和参数）。



### 认证方式

- **匿名用户模式**

　　最不安全的一种认证模式，任何人都无需密码验证而直接登录到FTP服务器  

- **本地用户模式**

　　通过Linux系统本地的账户密码认证的模式，相较于匿名更安全，配置也更简单。（如果有人通过FTP破解了账户的信息，就可以畅通无阻的登录到FTP服务器，从而控制整台服务器）  

- **虚拟用户模式**

　　更安全的一种认证模式，它需要FTP服务单独为客户建立数据库文件，虚拟出用来进行密码验证的账户信息，而这些账户信息在服务器系统中实际上是不存在的，仅供FTP服务进行认证使用。  




### 安装Vsftpd

当前环境为CentOS Linux 8.4，配置软件仓库

```bash
mv /etc/yum.repos.d/CentOS-Linux-BaseOS.repo /etc/yum.repos.d/CentOS-Linux-BaseOS.repo.bak
curl -o /etc/yum.repos.d/CentOS-Linux-BaseOS.repo https://mirrors.aliyun.com/repo/Centos-8.repo
dnf clean all
dnf makecache
```

安装vsftp软件包

```bash
dnf install vsftpd
```

备份原配置文件

```bash
mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
```

过滤#开头的行，生成新的配置文件

```bash
grep -v "#" /etc/vsftpd/vsftpd.conf.bak > /etc/vsftpd/vsftpd.conf
```

查看配置文件

```bash
cat /etc/vsftpd/vsftpd.conf

1  anonymous_enable=NO
2  local_enable=YES
3  write_enable=YES
4  local_umask=022
5  dirmessage_enable=YES
6  xferlog_enable=YES
7  connect_from_port_20=YES
8  xferlog_std_format=YES
9  listen=NO
10 listen_ipv6=YES
```

主配置文件的常用参数

| 参数                                               | 作用                                                         |
| -------------------------------------------------- | :----------------------------------------------------------- |
| listen=[YES\|NO]                                   | 是否以独立运行的方式监听服务                                 |
| listen_address=IP地址                              | 设置要监听的IP地址                                           |
| listen_port=21                                     | 设置FTP服务的监听端口                                        |
| download_enable＝[YES\|NO]                         | 是否允许下载文件                                             |
| userlist_enable=[YES\|NO]  userlist_deny=[YES\|NO] | 设置用户列表为“允许”还是“禁止”操作                           |
| max_clients=0                                      | 最大客户端连接数，0为不限制                                  |
| max_per_ip=0                                       | 同一IP地址的最大连接数，0为不限制                            |
| anonymous_enable=[YES\|NO]                         | 是否允许匿名用户访问                                         |
| anon_upload_enable=[YES\|NO]                       | 是否允许匿名用户上传文件                                     |
| anon_umask=022                                     | 匿名用户上传文件的umask值                                    |
| anon_root=/var/ftp                                 | 匿名用户的FTP根目录                                          |
| anon_mkdir_write_enable=[YES\|NO]                  | 是否允许匿名用户创建目录                                     |
| anon_other_write_enable=[YES\|NO]                  | 是否开放匿名用户的其他写入权限（包括重命名、删除等操作权限） |
| anon_max_rate=0                                    | 匿名用户的最大传输速率（字节/秒），0为不限制                 |
| local_enable=[YES\|NO]                             | 是否允许本地用户登录FTP                                      |
| local_umask=022                                    | 本地用户上传文件的umask值                                    |
| local_root=/var/ftp                                | 本地用户的FTP根目录                                          |
| chroot_local_user=[YES\|NO]                        | 是否将用户权限禁锢在FTP目录，以确保安全                      |
| local_max_rate=0                                   | 本地用户最大传输速率（字节/秒），0为不限制                   |

### 配置匿名用户模式

> vsftpd服务程序默认关闭了匿名登录的方式，我们需要开放匿名用户的上传、下载文件的权限，以及让匿名用户创建、删除、更名文件的权限。不建议在生产环境中使用此种模式

安装vsftpd软件包

```bash
dnf install vsftpd
```

备份原配置文件

```bash
mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
```

过滤#开头的行，生成新的配置文件

```bash
grep -v "#" /etc/vsftpd/vsftpd.conf.bak > /etc/vsftpd/vsftpd.conf
```

编辑配置文件

```
vim /etc/vsftpd/vsftpd.conf
```

```bash
anonymous_enable=YES
anon_umask=022
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES

local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
```

向匿名用户开放的权限参数以及作用

| 参数                        | 作用                               |
| --------------------------- | ---------------------------------- |
| anonymous_enable=YES        | 允许匿名访问模式                   |
| anon_umask=022              | 匿名用户上传文件的umask值          |
| anon_upload_enable=YES      | 允许匿名用户上传文件               |
| anon_mkdir_write_enable=YES | 允许匿名用户创建目录               |
| anon_other_write_enable=YES | 允许匿名用户修改目录名称或删除目录 |

创建在ftp默认访问目录下创建一个公用目录

```bash
mkdir  /var/ftp/pub
```

给与ftp用户子目录的属主

```bash
chown -R ftp /var/ftp/pub/
```

<font color=red> *注意：不建议给予父目录/var/ftp属主或777权限，会触发安全保护机制，导致用户无法登录*</font>

重启服务

```bash
systemctl restart vsftpd
```

加入开机自启

```bash
systemctl enable vsftpd
```

查看与FTP相关的SELinux域策略

```bash
getsebool -a | grep ftp
```

修改SELinux策略，-P永久生效

```bash
setsebool -P ftpd_full_access=on
```

允许firewall通过ftp协议

```bash
firewall-cmd --permanent --zone=public --add-service=ftp
```

重新加载firewall策略生效

```bash
firewall-cmd --reload
```



### 配置本地用户模式

> vsftpd服务程序默认关闭了匿名登录的方式，我们需要开放匿名用户的上传、下载文件的权限，以及让匿名用户创建、删除、更名文件的权限。不建议在生产环境中使用此种模式

安装vsftpd软件包

```bash
dnf install vsftpd
```

备份原配置文件

```bash
mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
```

过滤#开头的行，生成新的配置文件

```bash
grep -v "#" /etc/vsftpd/vsftpd.conf.bak > /etc/vsftpd/vsftpd.conf
```

编辑配置文件

```
vim /etc/vsftpd/vsftpd.conf
```

```bash
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
```
本地用户模式使用的权限参数以及作用

| 参数                | 作用                                              |
| ------------------- | ------------------------------------------------- |
| anonymous_enable=NO | 禁止匿名访问模式                                  |
| local_enable=YES    | 允许本地用户模式                                  |
| write_enable=YES    | 设置可写权限                                      |
| local_umask=022     | 本地用户模式创建文件的umask值                     |
| userlist_deny=YES   | 启用“禁止用户名单”，名单文件为ftpusers和user_list |
| userlist_enable=YES | 开启用户作用名单文件功能                          |

重启服务

```bash
systemctl restart vsftpd
```

加入开机自启

```bash
systemctl enable vsftpd
```

查看与FTP相关的SELinux域策略

```bash
getsebool -a | grep ftp
```

修改SELinux策略，-P永久生效

```bash
setsebool -P ftpd_full_access=on
```

允许firewall通过ftp协议

```bash
firewall-cmd --permanent --zone=public --add-service=ftp
```

重新加载firewall策略生效

```bash
firewall-cmd --reload
```

现在可以使用的本地账户，来登录ftp服务，但为了服务器的安全性，ftp服务默认禁止了大多本地系统用户的登录。如果想要使用root用户登录，删除ftp用户名单（ftpusers和user_list）中的root即可。

当然也可以创建一个不允许登录系统的账户，给ftp登录专用

```bash
useradd -s /sbin/nologin test
```

Ps：这里为什么需要两个同样的文件来禁用登录功能呢，玄机在与user_list上面的一段注释。如果主配置文件中的userlist_deny=YES则user_list变为黑名单，不允许用户登录（此为默认值）。如果主配置文件中的“userlist_deny=NO”则user_list变为白名单，允许名单内的用户登录。  



### 配置虚拟用户模式

安装vsftpd软件包

```bash
dnf install vsftpd
```

新建虚拟用户账号密码文件

```bash
cd /etc/vsftpd/
```

```bash
vim vuser.list
test1
pwd@test1
test2
pwd@test2
```

<font color=red>*注：奇数为用户名，偶数用户密码*</font>

使用哈希（hash）算法转换成数据库文件

```bash
db_load -T -t hash -f vuser.list vuser.db
```

降低此文件的权限

```bash
chmod 600 vuser.db
```

#建议删除明文信息文件（可选，怕忘密码可以不删）

```bash
rm -rf vuser.list
```

创建virtual本地用户用于和虚拟用户的映射，指定家目录，不允许该用户登录系统

```bash
useradd -d /var/ftproot -s /sbin/nologin virtual
```

<font color=red>*注：指定的家目录为/var/ftproot（上面的操作将设置此为ftp登录后默认进入的目录）*</font>

查看新生成的目录权限，为700

```bash
ls -ld /var/ftproot/
drwx------. 2 virtual virtual 62 Oct 19 10:44 /var/ftproot/
```

强制提升目录权限为755

```bash
chmod -Rf 755 /var/ftproot/
```

```bash
ls -ld /var/ftproot/
drwxr-xr-x. 2 virtual virtual 62 Oct 19 15:17 /var/ftproot/
```

建立支持虚拟用户的PAM[^1]文件


```bash
vim /etc/pam.d/vsftpd.vu
```


```bash
auth       required     pam_userdb.so db=/etc/vsftpd/vuser
account    required     pam_userdb.so db=/etc/vsftpd/vuser
```

备份原配置文件


```bash
mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
```

过滤#开头的行，生成新的配置文件

```bash
grep -v "#" /etc/vsftpd/vsftpd.conf.bak > /etc/vsftpd/vsftpd.conf
```

创建虚拟用户权限目录

```bash
mkdir /etc/vsftpd/vusers_dir
cd /etc/vsftpd/vusers_dir
```

不给予test1权限

```bash
touch test1
```

给与test2所有权限

```bash
vim test2
```

```bash
anon_upload_enable=YES
anon_mkdir_write_enable=YESbash
anon_other_write_enable=YES
```

编辑配置文件

```bash
vim /etc/vsftpd/vsftpd.conf
```

```bash
anonymous_enable=NO
local_enable=YES
write_enable=YES
guest_enable=YES
guest_username=virtual
allow_writeable_chroot=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd.vu
userlist_enable=YES
user_config_dir=/etc/vsftpd/vusers_dir
```

利用PAM文件进行认证时使用的参数以及作用

| 参数                       | 作用                                                        |
| -------------------------- | ----------------------------------------------------------- |
| anonymous_enable=NO        | 禁止匿名开放模式                                            |
| local_enable=YES           | 允许本地用户模式                                            |
| guest_enable=YES           | 开启虚拟用户模式                                            |
| guest_username=ftpvuser    | 指定虚拟用户账户                                            |
| pam_service_name=vsftpd.vu | 指定PAM文件                                                 |
| allow_writeable_chroot=YES | 允许对禁锢的FTP根目录执行写入操作，而且不拒绝用户的登录请求 |

重启服务

```bash
systemctl restart vsftpd
```

加入开机自启

```bash
systemctl enable vsftpd
```

查看与FTP相关的SELinux域策略

```bash
getsebool -a | grep ftp
```

修改SELinux策略，-P永久生效

```bash
setsebool -P ftpd_full_access=on
```

允许firewall通过ftp协议

```bash
firewall-cmd --permanent --zone=public --add-service=ftp
```

重新加载firewall策略生效

```bash
firewall-cmd --reload
```



### 连接测试

- test1只有下载权限
- test2拥有所有权限

安装lftp客户端工具，连接测试


```bash
dnf install lftp
```

```bash
#lftp的简易使用
lftp <FTP服务器IP> -u <用户名>	#登录FTP服务
lftp> ls						#列表显示当前所有内容
lftp> cd xxx					#进入指定目录
lftp> get xxx					#下载指定文件或目录到当前位置
lftp> put xxx					#上传指定文件或目录到FTP
```

FTP目录放入一个文件，测试使用

```bash
echo "hiahia" > /var/ftproot/lookup.txt
```

使用test1用户登录测试


```bash
[root@localhost ~]# lftp 192.168.88.133 -u test1
Password:                   
lftp test1@192.168.88.133:~> ls
-rw-r--r--    1 0        0               7 Oct 19 07:22 lookup.txt
lftp test1@192.168.88.133:/> get lookup.txt 		#能下载文件
7 bytes transferred
lftp test1@192.168.88.133:/> mkdir test1			#无法创建目录
mkdir: Access failed: 550 Permission denied. (test1)
lftp test1@192.168.88.133:/> ls
-rw-r--r--    1 0        0               7 Oct 19 07:22 lookup.txt
lftp test1@192.168.88.133:/> rm lookup.txt 			#无法删除文件
rm: Access failed: 550 Permission denied. (lookup.txt)
lftp test1@192.168.88.133:/> exit

[root@localhost ~]# ls			#查看下载到本地的文件，测试完毕无异常
anaconda-ks.cfg  lookup.txt
[root@localhost ~]# cat lookup.txt
hiahia
[root@localhost ~]# rm -rf lookup.txt
```

使用test2用户登录测试
```bash
[root@localhost ~]# lftp 192.168.88.133 -u test2
Password:                       
lftp test2@192.168.88.133:~> ls
-rw-r--r--    1 0        0               7 Oct 19 07:22 lookup.txt
lftp test2@192.168.88.133:/> get lookup.txt 			#能下载文件
7 bytes transferred
lftp test2@192.168.88.133:/> put /etc/vsftpd/vsftpd.conf			#能上传文件
338 bytes transferred
lftp test2@192.168.88.133:/> ls
-rw-r--r--    1 0        0               7 Oct 19 07:22 lookup.txt
-rw-------    1 1000     1000          338 Oct 19 07:43 vsftpd.conf
lftp test2@192.168.88.133:/> mkdir test2			#能创建目录
mkdir ok, `test2' created
lftp test2@192.168.88.133:/> ls
-rw-r--r--    1 0        0               7 Oct 19 07:22 lookup.txt
drwx------    2 1000     1000            6 Oct 19 07:43 test2
-rw-------    1 1000     1000          338 Oct 19 07:43 vsftpd.conf
lftp test2@192.168.88.133:/> rm lookup.txt 			#能删除文件
rm ok, `lookup.txt' removed
lftp test2@192.168.88.133:/> ls
drwx------    2 1000     1000            6 Oct 19 07:43 test2
-rw-------    1 1000     1000          338 Oct 19 07:43 vsftpd.conf
lftp test2@192.168.88.133:/> exit
[root@localhost ~]# ls 			#查看下载到本地的文件，测试完毕无异常
anaconda-ks.cfg  lookup.txt
[root@localhost ~]# cat lookup.txt
hiahia
```

Vsftpd服务程序登陆后所在目录

| 登录方式 | 默认目录             |
| -------- | -------------------- |
| 匿名公开 | /var/ftp             |
| 本地用户 | 该用户的家目录       |
| 虚拟用户 | 对应映射用户的家目录 |





## TFTP

 

### 简介

**TFTP（Trivial File Transfer Protocol, 简单文件传输协议）**，基于UDP协议。提供不复杂、开销小的文件传输服务，相当于FTP协议的简化版本。

   

TFTP，不需要客户端的权限认证，减少了带宽和系统的消耗。它的功能较少，甚至不能遍历目录，安全性也不如FTP。但因此在传输琐碎（Trivial）小文件时，效率更高。





### 安装TFTP

来安装体验一下，**tftp-server** 提供服务程序 | **tftp**是用于连接的客户端工具 | **xinetd[^2]**是管理程序的服务

   

```
dnf install tftp-server tftp xinted
```

 

安装TFTP软件包后，还需要再xinetd服务程序中将其开启，再RHEL 8 系统中，tftp所对应的配置文件默认不存在，需要根据示例文件（/usr/share/doc/xinetd/sample.conf）自行创建。把底下的配置直接复制进去就OK

 

```
vim /etc/xinetd.d/tftp
```

```
service tftp
{
        socket_type             = dgram
        protocol                = udp
        wait                    = yes
        user                    = root
        server                  = /usr/sbin/in.tftpd
        server_args             = -s /var/lib/tftpboot
        disable                 = no
        per_source              = 11
        cps                     = 100 2
        flags                   = IPv4
}
```

 

重启服务

```
systemctl restart tftp
```

开启自启

```
systemctl enable tftp
```

重启xinetd管理服务

```
systemctl restart xinetd
```

开启自启

```
systemctl enable xinetd
```

允许firewall通过udp/69

```
firewall-cmd --zone=public --permanent --add-port=69/udp
```

 重新加载firewall策略生效

```
firewall-cmd --reload
```

### 连接测试

新建测试文件，供下载

```
echo "keke" >> /var/lib/tftpboot/nn
```
TFTP的根目录为**/var/lib/tftpboot**，使用tftp客户端尝试连接

```
[root@localhost ~]# tftp 192.168.88.133

tftp> ls

?Invalid command (无法遍历目录)

tftp> get nn

tftp> quit

[root@localhost ~]# cat nn

keke
```

tftp命令中可用的参数以及作用

| 参数    | 作用                |
| ------- | ------------------- |
| ?       | 帮助信息            |
| put     | 上传文件            |
| get     | 下载文件            |
| verbose | 显示详细的处理信息  |
| status  | 显示当前的状态信息  |
| binary  | 使用二进制进行传输  |
| ascii   | 使用ASCII码进行传输 |
| timeout | 设置重传的超时时间  |
| quit    | 退出                |





[^1]: PAM（可插拔认证模块）是一种认证机制，通过一些动态链接库和统一的API把系统提供的服务与认证方式分开，使得系统管理员可以根据需求灵活调整服务程序的不同认证方式。通俗来讲，PAM是一组安全机制的模块，系统管理员可以用来轻易地调整服务程序的认证方式，而不必对应用程序进行任何修改。PAM采取了分层设计（应用程序层、应用接口层、鉴别模块层）的思想
[^2]: 在Linux系统中，TFTP服务是xinetd服务程序来管理的。xinetd服务可以用来管理多种轻量级的网络服务，而且具有强大的日志功能。它专门用于控制那些比较小的应用程序的开启和关闭，有点类似带有独立开关的插线板，想要开启哪个服务，就编辑对应的xinetd配置文件的开关参数。

