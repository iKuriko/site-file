---
title: "OpenVPN"
date: 2022-03-18T11:27:46+08:00
draft: true
description: 在CentOS8.5上部署OVPN，使用账号密码实现客户端登录
tags:
    - Linux
    - VPN
    - CentOS
---



## OVPN-Server部署



### 安装软件包

```bash
yum -y install openvpn easy-rsa    #ovpn服务端部署，CA证书生成和管理 
```

```bash
yum -y install -y lzo-devel openssl-devel pam-devel    #lzo压缩支持，openssl库支持，pam认证模块支持
```

 

### 生成 Server 证书

生成CA证书存放的目录，复制easy-rsa到/etc/openvpn目录下（也可以直接在原目录修改生成证书）

```bash
cp -r /usr/share/easy-rsa/ /etc/openvpn/easy-rsa
```

 

生成CA证书配置文件，复制easy-rsa配置文件到/etc/openvpn/easy-rsa/3.0.8目录下,并重命名为vars

```
cp -r /usr/share/doc/easy-rsa/vars.example /etc/openvpn/easy-rsa/3.0.8/vars
```

 

修改证书配置文件，其他的默认，主要是修改个人信息，也可以不改

```bash
vim /etc/openvpn/easy-rsa/3.0.8/vars
```

```bash
 95 set_var EASYRSA_REQ_COUNTRY   "CN"
 96 set_var EASYRSA_REQ_PROVINCE  "Beijing"
 97 set_var EASYRSA_REQ_CITY    "Beijing"
 98 set_var EASYRSA_REQ_ORG     "Center Certificate Co"
 99 set_var EASYRSA_REQ_EMAIL    "ikuriko@qq.com"
100 set_var EASYRSA_REQ_OU     "Personal Organization"
```



初始化环境

```bash
[root@play-1 ~]# cd /etc/openvpn/easy-rsa/3.0.8/

[root@play-1 3.0.8]# ./easyrsa init-pki

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa/3.0.8/vars
#注意：从：/etc/openvpn/Easy RSA/3.0.8/vars使用Easy RSA配置

init-pki complete; you may now create a CA or requests.
#初始化pki完成；您现在可以创建CA或请求。

Your newly created PKI dir is: /etc/openvpn/easy-rsa/3.0.8/pki
#您新创建的PKI目录是：/etc/openvpn/easy rsa/3.0.8/PKI
```



创建CA根证书，然后会提示设置密码，此处可用使用nopass参数选择不要密码,如果有密码服务器每次启动都要求输入密码

```bash
[root@play-1 3.0.8]# ./easyrsa build-ca nopass

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa/3.0.8/vars
#注意：在/etc/openvpn/easy-rsa/3.0.8/vars 使用Easy-RSA配置

Using SSL: openssl OpenSSL 1.1.1k FIPS 25 Mar 2021
#使用的SSL版本：openssl OpenSSL 1.1.1k

Generating RSA private key, 2048 bit long modulus (2 primes)
#生成RSA私钥
..............................................................................+++++
..................................+++++
e is 65537 (0x010001)
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:play-1
#通用名（例如：你的用户名，主机名或服务名）
CA creation complete and you may now import and sign cert requests.
#CA创建完成，您现在可以导入并签署证书请求。
Your new CA certificate file for publishing is at: 
/etc/openvpn/easy-rsa/3.0.8/pki/ca.crt
#要发布的新CA证书文件位于：/etc/openvpn/easy-rsa/3.0.8/pki/ca.crt
```



创建Server端的证书和私钥文件（根密钥）

 

```bash
[root@play-1 3.0.8]# ./easyrsa gen-req server nopass

Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa/3.0.8/vars

Using SSL: openssl OpenSSL 1.1.1k FIPS 25 Mar 2021
Generating a RSA private key
........+++++
..................+++++
writing new private key to '/etc/openvpn/easy-rsa/3.0.8/pki/easy-rsa-19567.mEo6g0/tmp.gK6LGc'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [server]:paly-1
#这里输入用户名、主机、或服务名
Keypair and certificate request completed. Your files are:
req: /etc/openvpn/easy-rsa/3.0.8/pki/reqs/server.req
key: /etc/openvpn/easy-rsa/3.0.8/pki/private/server.key
```

 

给Server端刚才申请的证书进行签名，提示confirm request details:时,输入yes

```bash
[root@play-1 3.0.8]# ./easyrsa sign server server
 
Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa/3.0.8/vars

Using SSL: openssl OpenSSL 1.1.1k FIPS 25 Mar 2021
You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.
Request subject, to be signed as a server certificate for 825 days:
 
subject=

  commonName        = paly-1
 
 
Type the word 'yes' to continue, or any other input to abort.
 Confirm request details: yes
 #这里输入yes进行确认
Using configuration from /etc/openvpn/easy-rsa/3.0.8/pki/easy-rsa-19603.18ogaV/tmp.rp74CT
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName      :ASN.1 12:'paly-1'
Certificate is to be certified until Jun 19 16:27:30 2024 GMT (825 days)
 
Write out database with 1 new entries
Data Base Updated
 
Certificate created at: /etc/openvpn/easy-rsa/3.0.8/pki/issued/server.crt
```

 

创建dh文件,秘钥交换算法

```bash
[root@play-1 3.0.8]# ./easyrsa gen-dh
 
Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa/3.0.8/vars
Using SSL: openssl OpenSSL 1.1.1k FIPS 25 Mar 2021
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
..................................+.......................................................................................................................................................................................................................................................................................................................................................................++*++*++*++*
 
DH parameters of size 2048 created at /etc/openvpn/easy-rsa/3.0.8/pki/dh.pem
```

 

创建tls认证所需的秘钥文件

```bash
[root@play-1 3.0.8]# openvpn --genkey --secret ta.key
```



查看当前生成的文件目录结构

```bash
[root@play-1 3.0.8]# tree pki

pki
├── ca.crt
├── certs_by_serial
│  └── EFD5BE86E8C4AF3D921EF8994E7B88C0.pem
├── dh.pem
├── index.txt
├── index.txt.attr
├── index.txt.attr.old
├── index.txt.old
├── issued
│  └── server.crt
├── openssl-easyrsa.cnf
├── private
│  ├── ca.key
│  └── server.key
├── renewed
│  ├── certs_by_serial
│  ├── private_by_serial
│  └── reqs_by_serial
├── reqs
│  └── server.req
├── revoked
│  ├── certs_by_serial
│  ├── private_by_serial
│  └── reqs_by_serial
├── safessl-easyrsa.cnf
├── serial
└── serial.old
 
12 directories, 15 files
```



新建证书管理目录，拷贝证书相关文件到该目录下

```bash
[root@play-1 3.0.8]# mkdir /etc/openvpn/certs
[root@play-1 3.0.8]# cp ./pki/ca.crt /etc/openvpn/certs
[root@play-1 3.0.8]# cp ./pki/dh.pem /etc/openvpn/certs/
[root@play-1 3.0.8]# cp ./pki/issued/server.crt /etc/openvpn/certs/
[root@play-1 3.0.8]# cp ./pki/private/server.key /etc/openvpn/certs/
[root@play-1 3.0.8]# cp ta.key /etc/openvpn/certs
```

 

### Server 配置

拷贝模板配置文件到OVPN工作目录

```bash
cp /usr/share/doc/openvpn/sample/sample-config-files/server.conf /etc/openvpn/
```

 

修改后的配置文件如下，方便复制，详细的参数介绍放在后面

```bash
[root@play-1 openvpn]# cat server.conf | grep -vE '^#|^;|^$'
```

```bash
local 0.0.0.0
port 55003
proto udp
dev tap
ca /etc/openvpn/certs/ca.crt
cert /etc/openvpn/certs/server.crt
key /etc/openvpn/certs/server.key
dh /etc/openvpn/certs/dh.pem
server 100.88.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 114.114.114.114"
client-config-dir /etc/openvpn/ccd
client-to-client
duplicate-cn
keepalive 10 120
tls-auth /etc/openvpn/certs/ta.key 0
cipher AES-256-GCM
comp-lzo
max-clients 100
user openvpn
group openvpn
persist-key
persist-tun
status openvpn-status.log
log /etc/openvpn/log/openvpn.log
log-append /etc/openvpn/log/openvpn.log
verb 3
mute 20
tls-version-min 1.3
client-cert-not-required
auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env
script-security 3
username-as-common-name
```

 

配置文件的参数介绍

```bash
[root@play-1 openvpn]# cat /etc/openvpn/server.conf 

#监听的本地IP地址
local 0.0.0.0

#监听的端口号
port 55003

#指定采用的传输协议，tcp|udp
proto udp

#指定创建的通信隧道类型，tun|tap|，windows服务器必须是tap
dev tap

#指定CA证书的文件路径
ca /etc/openvpn/certs/ca.crt

#指定服务器端的证书文件路径
cert /etc/openvpn/certs/server.crt

#指定服务器端的私钥文件路径，这个文件应该保密
key /etc/openvpn/certs/server.key 

#指定迪菲赫尔曼参数的文件路径
dh /etc/openvpn/certs/dh.pem

#指定隧道占用的IP地址段和子网掩码，不能和服务器上的其他网段相同
server 100.88.0.0 255.255.255.0

 
#服务器自动给客户端分配IP后，客户端下次连接时，仍然采用上次的IP地址(第一次 分配的IP保存在ipp.txt中，下一次分配其中保存的IP)。
ifconfig-pool-persist ipp.txt

#自动推送客户端上的网关及DHCP,此项开启了流量转发,有这项才能使用服务器代理上 网
push "redirect-gateway def1 bypass-dhcp"

#OpenVPN的DHCP功能为客户端提供指定的 DNS、WINS 等
push "dhcp-option DNS 114.114.114.114"

#客户端的配置下发目录
client-config-dir /etc/openvpn/ccd

#允许客户端与客户端相连接，默认情况下客户端只能与服务器相连接
client-to-client

#允许通过客户端证书多次登录，看需求配置
duplicate-cn

#每10秒ping一次，连接超时时间设置为120秒
keepalive 10 120

#开启TLS-auth，使用ta.key防御攻击。服务器端的第二个参数值为0，客户端的为1。
tls-auth /etc/openvpn/certs/ta.key 0

#加密认证算法,2.4之前是AES-256-CBC
cipher AES-256-GCM

#使用lzo压缩的通讯,服务端和客户端都必须配置
comp-lzo

#最大连接用户
max-clients 100

#定义运行的用户和组,openvpn用户是安装的时候系统自动创建的
user openvpn
group openvpn

#重启时仍保留一些状态
persist-key
persist-tun

#输出短日志,每分钟刷新一次,以显示当前的客户端
status openvpn-status.log

#日志保存路径
log /etc/openvpn/log/openvpn.log
log-append /etc/openvpn/log/openvpn.log

#指定日志文件的记录详细级别，可选0-9，等级越高日志内容越详细
verb 3

#相同信息的数量，如果连续出现 20 条相同的信息，将不记录到日志中
mute 20

#下面这项只能udp连接开启
explicit-exit-notify 1

#设置tls最低版本为1.3,连接的客户端如果是2.4以下则配置为1.0
tls-version-min 1.3

#不需要客户证书
client-cert-not-required

#客户账号密码验证脚本
auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env

#设置脚本安全等级 允许用户自定义脚本使用
script-security 3

#用户名作为通用名（别名）
username-as-common-name
```

 

### 配置账号密码登录

用户使用账号密码登录认证时使用的脚本（来自OpenVPN官网）

```bash
[root@play-1 openvpn]# vim checkpsw.sh 
```

```sh
#!/bin/sh 
###########################################################  
# checkpsw.sh (C) 2004 Mathias Sundman <mathias@openvpn.se> 
#  
# This script will authenticate OpenVPN users against  
# a plain text file. The passfile should simply contain  
# one row per user with the username first followed by  
# one or more space(s) or tab(s) and then the password.  
 
PASSFILE="/etc/openvpn/psw-file"          
LOG_FILE="/etc/openvpn/openvpn-password.log" 
TIME_STAMP=`date "+%Y-%m-%d %T"`  
 
###########################################################  

if [ ! -r "${PASSFILE}" ]; then  
	  echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >> ${LOG_FILE}  
	    exit 1  
    fi  
    CORRECT_PASSWORD=`awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $2;exit}' ${PASSFILE}`  
    if [ "${CORRECT_PASSWORD}" = "" ]; then   
	      echo "${TIME_STAMP}: User does not exist: username=\"${username}\", password=  
	      \"${password}\"." >> ${LOG_FILE}  
	        exit 1  
	fi  
	if [ "${password}" = "${CORRECT_PASSWORD}" ]; then   
		  echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}  
		    exit 0  
	    fi  
	    echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=  
	    \"${password}\"." >> ${LOG_FILE} 

```



新建用户账号密码记录文件，后续添加账号可直接添加

```bash
[root@play-1 openvpn]# vim psw-file
test	123456			# 用户名：test，密码：123456。IP地址：100.88.0.254
```



 新建登录密码验证日志文件

```bash
touch /etc/openvpn/openvpn-password.log
```

 

 设置以上两个文件的所属组和所属用户为openvpn

```bash
chown openvpn:openvpn checkpsw.sh 
chown openvpn:openvpn openvpn-password.log
```

 

 新建用户下发配置目录

```bash
mkdir /etc/openvpn/ccd 
```

 

可以给test用户分配一个固定的IP地址

```bash
vim /etc/openvpn/ccd/test 
```

```bash
ifconfig-push 100.88.0.254 255.255.255.0
```



### 服务启动

因为CentOS8没有OpenVPN的unit服务启动文件，需要自己新建

```bash
vim /lib/systemd/system/openvpn@.service
```

```bash
[Unit]
Description=OpenVPN Robust And Highly Flexible Tunneling Application On %I
After=network.target

[Service]
Type=notify
PrivateTmp=true
ExecStart=/usr/sbin/openvpn --cd /etc/openvpn/ --config %i.conf

[Install]
WantedBy=multi-user.target
```

重新加载服务的启动配置文件

```bash
systemctl daemon-reload
```

启动OVPN-Server服务

```bash
systemctl start openvpn@server
```

将OVPN-Server服务设置为开机自启

```bash
systemctl enable openvpn@server
```

 首先查看服务状态

```bash
systemctl status openvpn@server
```

查看端口和进程是否启动成功

```bash
ss -utpln |grep openvpn
```

```bash
ps -aux|grep openvpn
```

查看日志

```bash
tail -f /etc/openvpn/openvpn-status.log
```



## OVPN-Client 拨号

### 安装软件包

```bash
yum -y install openvpn easy-rsa    #ovpn服务端部署，CA证书生成和管理 
```

```bash
yum -y install -y lzo-devel openssl-devel pam-devel    #lzo压缩支持，openssl库支持，pam认证模块支持
```

 



### 生成 Client 证书

进入证书管理目录

```bash
cd /etc/openvpn/easy-rsa/3.0.8
```

生成客户端1的证书，客户端2依次类推。如果配置 Server 使用账号密码认证登录，则无需申请多个证书，所有客户端可共用同一个证书实现认证登录，但是需要在 server.conf 加入 duplicate-cn 的配置选项

```bash
[root@play-1 3.0.8]# ./easyrsa gen-req client nopass
 
Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa/3.0.8/vars
Using SSL: openssl OpenSSL 1.1.1k FIPS 25 Mar 2021
Generating a RSA private key
...........+++++
....................................................................................+++++
writing new private key to '/etc/openvpn/easy-rsa/3.0.8/pki/easy-rsa-21281.oB04fP/tmp.QEXXEd'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [client]:play-2 
#这里输入客户端的用户名，主机或服务名
 
Keypair and certificate request completed. Your files are:
req: /etc/openvpn/easy-rsa/3.0.8/pki/reqs/client.req
key: /etc/openvpn/easy-rsa/3.0.8/pki/private/client.key
```

 

为客户端的证书进行签名，输入yes进行确认

```bash
[root@play-1 3.0.8]# ./easyrsa sign client client
 
Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa/3.0.8/vars
Using SSL: openssl OpenSSL 1.1.1k FIPS 25 Mar 2021
 
 
You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.
 
Request subject, to be signed as a client certificate for 825 days:
 
subject=
  commonName        = play-2
 
 
Type the word 'yes' to continue, or any other input to abort.
 Confirm request details: yes
 #这里输入yes进行确认
Using configuration from /etc/openvpn/easy-rsa/3.0.8/pki/easy-rsa-21325.18TS67/tmp.iF9iA2
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName      :ASN.1 12:'play-2'
Certificate is to be certified until Jun 19 17:25:07 2024 GMT (825 days)
 
Write out database with 1 new entries
Data Base Updated
 
Certificate created at: /etc/openvpn/easy-rsa/3.0.8/pki/issued/client.crt
```

 

拷贝客户证书存放到Client目录

```bash
[root@play-1 3.0.8]# cp ./pki/issued/client.crt /etc/openvpn/client/
[root@play-1 3.0.8]# cp ./pki/private/client.key /etc/openvpn/client/
```





### Client 配置

将服务端的client.crt（客户端证书）、client.key（客户端密钥）、ta.key（tls认证所需的秘钥文件）、ca.crt（CA根证书）四个文件拷贝到本地客户端目录的/etc/openvpn目录下（Windows 客户端为config目录）

```bash
[root@play-2 ~]# ls /etc/openvpn/certs/
ca.crt  client.crt  client.key  ta.key
```



配置客户端配置文件,拷贝客户端sample-config目录下的client.conf文件到config目录下

```bash
cp /usr/share/doc/openvpn/sample/sample-config-files/client.conf /etc/openvpn
```



修改后的配置文件如下，方便复制，详细的参数介绍放在后面

```bash
[root@play-1 openvpn]# cat /etc/openvpn/client.conf | grep -Ev "^#|^;|^$"

client
dev tap
proto udp
remote 192.168.88.133 55003
nobind
persist-key
persist-tun
ca /etc/openvpn/certs/ca.crt
cert /etc/openvpn/certs/client.crt
key /etc/openvpn/certs/client.key
remote-cert-tls server
cipher AES-256-GCM
tls-auth /etc/openvpn/certs/ta.key 1
comp-lzo 
verb 3
mute 20
tls-version-min 1.3
auth-nocache
auth-user-pass
ncp-disable
auth SHA1
tls-client
resolv-retry infinite
```

 配置文件的参数介绍

```bash
[root@play-2 ~]# cat /etc/openvpn/client.conf 
```

```bash
#客户端
client

#隧道类型，与服务器一致
dev tap

#tcp|udp，与服务器一致
proto udp

#服务器ip和端口
remote 192.168.88.133 55003

#不绑定本地特定的端口
nobind

#服务器重启后保持的一些状态
persist-key
persist-tun

#客户端证书目录
ca /etc/openvpn/certs/ca.crt
cert /etc/openvpn/certs/client.crt
key /etc/openvpn/certs/client.key

#远程证书验证
remote-cert-tls server

#加密方式（BF-CBC）
cipher AES-256-GCM

#tls握手秘钥,与服务器保持一致,服务器0,客户端1
tls-auth /etc/openvpn/certs/ta.key 1

#开启数据压缩
comp-lzo 

#日志级别
verb 3

#相同信息的数量，如果连续出现 20 条相同的信息，将不记录到日志中
mute 20

#tls最低版本,与服务器保持一致
tls-version-min 1.3

#不保存密码
auth-nocache

#使客户端中所有流量经过VPN,所有网络连接都使用vpn
redirect-gateway def1

#用户需要密码认证
auth-user-pass

#关闭网络控制协议
ncp-disable

#哈希加密认证
auth SHA1

#客户端开启tls加密
tls-client

#重新获取DNS
resolv-retry infinite
```

 

### 配置账号密码登录

```
vim /etc/openvpn/passwd
```

```bash
test    #第一行用户名
123456    #第二行密码
```



### 服务启动

```bash
openvpn --daemon --cd /etc/openvpn --config client.conf --auth-user-pass /etc/openvpn/passwd --log-append /var/log/openvpn.log
```

 

参数详解：

--daemon    #以后台守护进程的方式执行

--cd /etc/openvpn    #进入openvpn工作目录

--config client.conf    #指定拨号使用的配置文件

--auth-user-pass /etc/openvpn/passwd    #指定认证使用的用户名和密码文件

--log-append /var/log/openvpn.log    #指定记录openvpn日志的文件位置

 

 ## OVPN-Server 代理上网



想要将OVPN-Server当作网关路由器上网使用。首先，你的OVPN-Server需要拥有一个能够访问目标网段地址的网络接口，这个接口获取IP地址的方式可以是绑定的静态IP，也可以是DHCP动态获取。

### 路由转发

修改内核参数，开启基于ipv4的路由转发功能

```bash
echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf
```

 立即生效配置

```
sysctl -p
```

### 防火墙流量转发

关闭firewall防火墙

```bash
systemctl stop firewalld
systemctl disable firewalld
```

关闭selinux，有时候会碍事

```bash
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config     #永久关闭，重启生效
setenforce 0    #临时关闭
```

启动iptables，并设置开机自启

```bash
systemctl start iptables
systemctl enable iptables
```

 

配置SNAT地址转换，将来自OVPN隧道网段的IP地址转换为本机eth0的IP地址，代理上网主要以iptables转发流量实现。

```bash
iptables -t nat -A POSTROUTING -s 100.88.0.0/24 -j SNAT --to-source 192.168.88.133    #适用于出口为静态IP地址
iptables -t nat -A POSTROUTING -s 100.88.0.0/24 -o eth0 -j MASQUERADE    #适用于出口为动态IP地址
```



配置转发策略，FORWARD链，允许转发

```bash
iptables -P FORWARD ACCEPT
```



配置进站策略，INPUT链，允许tcp/udp协议55003端口通过防火墙

```bash
iptables -I INPUT -p tcp --dport 55003 -m comment --comment "openvpn" -j ACCEPT
iptables -I INPUT -p udp --dport 55003 -m comment --comment "openvpn" -j ACCEPT
```



保存防火墙规则，并重启

```bash
service iptables save
systemctl restart iptables
```



---



References & Resources：

[centos8安装配置openvpn实现服务器代理上网](https://zhuanlan.zhihu.com/p/429566474)



 

 

 
