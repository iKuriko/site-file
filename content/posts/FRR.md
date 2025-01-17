---
title: "FRR"
date: 2025-01-17T16:53:19+08:00
draft: true
tags:
  - CentOS Linux
  - Linux services
  - Network tools
description: FRRouting 是 Linux 上一款开源的路由模拟器工具，它能使你的 Linux 主机变身为思科路由器
---

## 安装

```bash
#安装软件源
FRRVER="frr-stable"
curl -O https://rpm.frrouting.org/repo/$FRRVER-repo-1-0.el8.noarch.rpm
yum install -y ./$FRRVER*
```

```bash
#安装frr
yum install -y frr frr-pythontools
```

```bash
#卸载软件源
#yum remove -y frr-stable-repo
#rm /etc/yum.repos.d/frr-stable-repo-1-0.el8.noarch.rpm
```

## 升级

CentOS8.5.2111 版本需要安装最新的 lib 库以支持安装最新版本的frr

```bash
#安装lib库
wget https://dl.rockylinux.org/pub/rocky/8/AppStream/x86_64/os/Packages/j/json-c-devel-0.13.1-3.el8.x86_64.rpm
wget https://dl.rockylinux.org/pub/rocky/8/Devel/x86_64/os/Packages/j/json-c-0.13.1-3.el8.x86_64.rpm
yum -y localinstall json-c-0.13.1-3.el8.x86_64.rpm json-c-devel-0.13.1-3.el8.x86_64.rpm
```

```bash
#安装frr
yum update -y frr frr-pythontools
```

## 启动

```bash
#启动 frr 对各种路由协议和名字空间的支持
sed -i "s/bgpd=no/bgpd=yes/g" /etc/frr/daemons
sed -i "s/isisd=no/isisd=yes/g" /etc/frr/daemons
sed -i "s/bfdd=no/bfdd=yes/g" /etc/frr/daemons
sed -i "s/isisd=no/isisd=yes/g" /etc/frr/daemons
sed -i "s/ospfd=no/ospfd=yes/g" /etc/frr/daemons
sed -i "s/-A 127.0.0.1 -s 90000000/-A 127.0.0.1 -n -s 90000000/g" /etc/frr/daemons
sed -i "s/##watchfrr_options=\"--netns\"/watchfrr_options=\"--netns\"/g" /etc/frr/daemons
```

```bash
systemctl start frr     #启动服务
```

```bash
systemctl enable frr    #开机自启
```



## 常用命令

### #vtysh		#进入模拟器命令行

```bash
configure terminal 					  			#进入全局配置

show running-config    							#查看所有配置

show ip route     								#查看路由表

show ip route vrf vrf1    						#查看vrf1路由表

ip route 115.115.115.0/24 12.1.1.2 vrf vrf1     #宣告路由vrf1的路由（全局模式静态路由）

end                                             #回到特权模式

touch /etc/frr/frr.conf							#创建保存配置的文件（如有需要）

copy /etc/frr/frr.conf running-config			#加载配置

copy running-config startup-config & w		    #保存配置
```



## #IS-IS

```bash
router isis vrf1 vrf vrf1    					#在vrf1 下启用isis，命名为vrf1

net 49.0001.1111.1111.1111.1111.00    			#配置NSAP地址，区域号

lsp-mtu 1400    								#配置lsp-mtu

exit    										#退出vrf1

interface gretap1 vrf vrf1   		 			#进入到命名空间vrf1的gretap1接口下

ip route isis vrf1    							#启用isis功能

isis metric 80    								#配置优先级，默认vPE-vPE的不设置metric,vPE-cPE的设置metric

redistribute ipv4 static level-1    			#isis重定向级别1

redistribute ipv4 static level-2    			#isis重定向级别2

show isis vrf vrf1 route    					#查看vrf1的isis路由表

show isis vrf vrf1 neighbor    					#查看vrf1的邻居关系表
```



## #BGP

```bash
router bgp 64931								#启用bgp AS号：64931

no bgp ebgp-requires-policy 					#关闭EBGP需要策略连接（如果不关闭，EBGP邻居之间由于没有策略不允许连接）

neighbor 100.0.10.1 remote-as 64939      		#指定邻居IP和AS号

neighbor 100.0.10.1 bfd							#对邻居启用BFD，需要在cat /etc/frr/daemons开启BFD的支持

neighbor 100.0.10.1 shutdown					#关闭邻居

address-family ipv4 unicast						#IPv4单播地址簇

network 10.80.0.0/16							#宣告网络，发布路由给邻居

redistribute connected							#重定向发布直连网段到BGP

redistribute static								#重定向发布静态路由到BGP（这里的静态指手动在交换机模式下配置的路由，名字空间里的路由是kernel路由）

redistribute static								#重定向发布内核路由到BGP

neighbor 192.0.0.26 next-hop-self				#邻居的下一跳指向自己（IBGP邻居需要配置此条）

exit-address-family								#退出

show ip bgp vrf ns1 summary					    #查看bgp邻居状态

show ip bgp vrf ns1							    #查看bgp路由

ip bgp neighbors 198.18.224.214					#查看bgp邻居详细信息
```



### #weight（BGP属性）

```bash
configure terminal

router bgp 64931

neighbor 100.0.10.1 remote-as 64930

address-family ipv4 unicast

neighbor 100.0.10.1 weight 120					#给邻居设置weight(权重值)。默认权重为0，权重越大，路由越优先
```



### #route-map（BGP属性）

```bash
configure terminal

route-map routemap1 permit 10				    #创建routemap规则为允许

set as-path prepend 64901 64902 64903			#设置as-path，在已有的路径后面追加三个as

router bgp 64931

address-family ipv4 unicast

neighbor 100.0.10.1 route-map routemap1 out		#应用route-map给邻居，动作为out，意思是出方向策略
```



### #ip prefix-list（BGP属性）

```bash
configure terminal

ip prefix-list prefixlist1 seq 5 permit 198.18.1.0/24	#创建ip prefix-list 允许网段198.18.1.0/24通过

ip prefix-list prefixlist1 seq 10 deny any				#拒绝其他所有网段

router bgp 64931

address-family ipv4 unicast

neighbor 198.18.1.2 prefix-list prefixlist1 out			#应用ip prefix-list给邻居，动作为out，意思是出方向策略
```



