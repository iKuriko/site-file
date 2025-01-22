---
title: "Dig"
date: 2023-11-07T16:21:11+08:00
draft: true
tags:
  - Tools
description: 非常好用的DNS查询工具
---



**dig 命令由 bind-utils 软件包提供**

```bash
yum install bind-utils -y
```



**使用dig查询某个域名**

```bash
dig www.baidu.com
```

```
#dig www.baidu.com

; <<>> DiG 9.11.26-RedHat-9.11.26-6.el8 <<>> www.baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14229
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;www.baidu.com.			IN	A

;; ANSWER SECTION:
www.baidu.com.		125	IN	CNAME	www.a.shifen.com.
www.a.shifen.com.	124	IN	A	180.101.50.242
www.a.shifen.com.	124	IN	A	180.101.50.188

;; Query time: 25 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: 二 11月 07 16:23:37 CST 2023
;; MSG SIZE  rcvd: 104

```



**dig 命令默认的输出信息比较丰富，大概可以分为 5 个部分。**

第一部分显示 dig 命令的版本和输入的参数。

第二部分显示服务返回的一些技术详情，比较重要的是 status。如果 status 的值为 NOERROR 则说明本次查询成功结束。

第三部分中的 "QUESTION SECTION" 显示我们要查询的域名。

第四部分的 "ANSWER SECTION" 是查询到的结果。

第五部分则是本次查询的一些统计信息，比如用了多长时间，查询了哪个 DNS 服务器，在什么时间进行的查询等等。

默认情况下 dig 命令查询 A 记录，上图中显示的 A 即说明查询的记录类型为 A 记录。在尝试查询其它类型的记录前让我们先来了解一下常见的 DNS 记录类型。

 

**常见 DNS 记录的类型**

| 类型  | 作用                                                         |
| ----- | ------------------------------------------------------------ |
| A     | 地址记录，用来指定域名的 IPv4 地址，如果需要将域名指向一个 IP 地址，就需要添加 A 记录 |
| AAAA  | 用来指定主机名或域名对应的 IPv6 地址记录                     |
| CNAME | 如果需要将域名指向另一个域名，再由另一个域名提供 IP 地址，就需要添加 CNAME 记录 |
| MX    | 如果需要设置邮箱，让邮箱能够收到邮件，需要添加 MX 记录       |
| NS    | 域名服务器记录，如果需要把子域名交给其他 DNS 服务器解析，就需要添加 NS 记录 |
| SOA   | SOA 记录是所有区域性文件中的强制性记录，它必须是一个文件中的第一个记录 |
| TXT   | 可以写任何东西，长度限制为 255。绝大多数的 TXT 记录是用来做 SPF 记录（反垃圾邮件） |



**查询某个类型的记录**

```bash
dig www.baidu.com CNAME
```

```
#dig www.baidu.com CNAME

; <<>> DiG 9.11.26-RedHat-9.11.26-6.el8 <<>> www.baidu.com CNAME
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49689
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;www.baidu.com.			IN	CNAME

;; ANSWER SECTION:
www.baidu.com.		848	IN	CNAME	www.a.shifen.com.

;; Query time: 20 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: 二 11月 07 16:34:54 CST 2023
;; MSG SIZE  rcvd: 72

```



从指定的DNS服务器上查询

```bash
dig @223.5.5.5 www.baidu.com
```

```
#dig @223.5.5.5 www.baidu.com

; <<>> DiG 9.11.26-RedHat-9.11.26-6.el8 <<>> @223.5.5.5 www.baidu.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58549
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1408
;; QUESTION SECTION:
;www.baidu.com.			IN	A

;; ANSWER SECTION:
www.baidu.com.		24	IN	CNAME	www.a.shifen.com.
www.a.shifen.com.	24	IN	A	110.242.68.3
www.a.shifen.com.	24	IN	A	110.242.68.4

;; Query time: 4 msec
;; SERVER: 223.5.5.5#53(223.5.5.5)
;; WHEN: 二 11月 07 16:38:09 CST 2023
;; MSG SIZE  rcvd: 101

```

 

**反向解析域名**

```bash
dig -x 223.5.5.5 +short
```

```
# dig -x 223.5.5.5 +short
public1.alidns.com.
```

详细输出

```bash
dig -x 223.5.5.5
```

```
# dig -x 223.5.5.5

; <<>> DiG 9.11.26-RedHat-9.11.26-6.el8 <<>> -x 223.5.5.5
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38486
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;5.5.5.223.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
5.5.5.223.in-addr.arpa.	1402	IN	PTR	public1.alidns.com.

;; Query time: 27 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: 二 11月 07 16:44:10 CST 2023
;; MSG SIZE  rcvd: 83

```

 

**跟踪整个DNS查询过程**

```bash
dig +trace www.baidu.com
```

```
# dig +trace www.baidu.com

; <<>> DiG 9.11.26-RedHat-9.11.26-6.el8 <<>> +trace www.baidu.com
;; global options: +cmd
.			364	IN	NS	i.root-servers.net.
.			364	IN	NS	h.root-servers.net.
.			364	IN	NS	a.root-servers.net.
.			364	IN	NS	l.root-servers.net.
.			364	IN	NS	e.root-servers.net.
.			364	IN	NS	b.root-servers.net.
.			364	IN	NS	d.root-servers.net.
.			364	IN	NS	f.root-servers.net.
.			364	IN	NS	g.root-servers.net.
.			364	IN	NS	c.root-servers.net.
.			364	IN	NS	k.root-servers.net.
.			364	IN	NS	j.root-servers.net.
.			364	IN	NS	m.root-servers.net.
;; Received 251 bytes from 114.114.114.114#53(114.114.114.114) in 22 ms

com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
;; Received 1173 bytes from 199.9.14.201#53(b.root-servers.net) in 153 ms

baidu.com.		172800	IN	NS	ns2.baidu.com.
baidu.com.		172800	IN	NS	ns3.baidu.com.
baidu.com.		172800	IN	NS	ns4.baidu.com.
baidu.com.		172800	IN	NS	ns1.baidu.com.
baidu.com.		172800	IN	NS	ns7.baidu.com.
;; Received 849 bytes from 192.48.79.30#53(j.gtld-servers.net) in 179 ms

www.baidu.com.		1200	IN	CNAME	www.a.shifen.com.
;; Received 100 bytes from 36.155.132.78#53(ns3.baidu.com) in 23 ms

```

