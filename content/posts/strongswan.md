---
title: "Strongswan VPN"
date: 2024-12-20T17:51:41+08:00
draft: true
tags:
  - Linux services
  - VPN
description: Linux 下的 IKEv2/IPSEC VPN
---



## 安装



```bash
#下载软件包
wget https://download.strongswan.org/strongswan-5.9.14.tar.gz
```

```bash
#解压软件包
tar -zxvf strongswan-5.9.14.tar.gz
```

```bash
#进入目录
cd strongswan-5.9.14/
```

```bash
#安装依赖的软件包
yum install -y libpam0g-dev libssl-dev make gcc curl gmp-devel openssl-devel pam-devel 
```

```bash
#创建安装目录
mkdir /usr/local/strongswan/
```

```bash
#配置安装参数
./configure  --enable-eap-identity --enable-eap-md5 --enable-eap-mschapv2 --enable-eap-tls --enable-eap-ttls --enable-eap-peap  --enable-eap-tnc --enable-eap-dynamic --enable-eap-radius --enable-xauth-eap  --enable-xauth-pam  --enable-dhcp  --enable-openssl  --enable-addrblock --enable-unity  --enable-certexpire --enable-radattr --enable-swanctl --enable-openssl --disable-gmp --enable-kernel-libipsec --prefix=/usr/local/strongswan/
```

```bash
#编译安装
make && make install
```





## 配置



```bash
#安装完成进入工作目录
cd /usr/local/strongswan
```

```bash
#新建配置文件（在5.8版本之前，strongswan 默认使用 ipsec.conf 配置文件，之后改用 swanctl.conf 配置）
vim etc/swanctl/conf.d/你的域名.conf
```

```bash
connections {
    ikev2-eap-mschapv2 {
        version = 2
        unique = never
        rekey_time = 0s
        fragmentation = yes
        dpd_delay = 60s
        send_cert = always
        # pools = ipv4-addrs, ipv6-addrs
        pools = ipv4-addrs
        proposals = aes256-sha256-modp2048, aes256-sha256-prfsha256-modp2048, aes256gcm16-prfsha384-modp1024, default
        local_addrs = %any
        local {
            certs = cert.pem
            id = 你的域名    # 此处填入你自己的域名
        }
        remote {
            auth = eap-mschapv2
            eap_id = %any
        }
        children {
            ikev2-eap-mschapv2 {
                # remote_ts = 123.86.130.0/0
                # local_ts = 0.0.0.0/0,::/0
                local_ts = 223.5.5.5   # VPN目标的地址
                rekey_time = 0s
                dpd_action = clear
                esp_proposals = aes256-sha256, aes128-sha1, aes256-sha256-modp2048-modpnone, default
                # dh_group = modp2048  # 添加此行以指定DH组
            }
        }
    }
}
pools {
    ipv4-addrs {
        addrs = 123.86.130.0/24    # 分配给隧道的地址池
        # addrs = 10.10.0.0/24
        # dns = 8.8.8.8,223.5.5.5
    }
    #ipv6-addrs {
    #    addrs = fec1::0/24
        # dns = 2001:4860:4860::8888,2606:4700:4700::1111
    #}
}
secrets {
    private-xxx {
        file = privkey.pem
    }   
    eap-camce {
        id = user    #拨号使用的用户名
        secret = "passwd"    #拨号使用的密码
    }
    #eap-camce {
    #    id = user2    #拨号使用的用户名
    #    secret = "passwd2"    #拨号使用的密码
    #}
}

```



```bash
#安装证书(此处为使用acme.sh申请的免费SSL证书，详情看另一篇文章：Acme.sh Aliyun)
ln -s /root/.acme.sh/你的域名/你的域名.key /usr/local/strongswan/etc/swanctl/private/privkey.pem 
ln -s /root/.acme.sh/你的域名/fullchain.cer /usr/local/strongswan/etc/swanctl/x509/cert.pem 
ln -s /root/.acme.sh/你的域名/ca.cer /usr/local/strongswan/etc/swanctl/x509ca/ca.pem 
```



```bash
#如果是用letsencrypt申请的证书，则使用以下命令
ln -s /etc/letsencrypt/live/你的域名/privkey.pem /usr/local/strongswan/etc/swanctl/private/privkey.pem
ln -s /etc/letsencrypt/live/你的域名/cert.pem /usr/local/strongswan/etc/swanctl/x509/cert.pem
ln -s /etc/letsencrypt/live/你的域名/chain.pem /usr/local/strongswan/etc/swanctl/x509ca/ca.pem
```



## 启动

```bash
#启动服务
/usr/local/strongswan/sbin/ipsec start 
```

```bash
#加载配置
/usr/local/strongswan/sbin/swanctl --load-all --noprompt
```



## 测试

PC客户端使用自带的VPN工具连接

![PC1.png](/images/swan/PC1.png)

![PC2.png](/images/swan/PC2.png)



手机客户端下载strongswan app

![Phone1.png](/images/swan/Phone1.png)

![Phone2.png](/images/swan/Phone2.png)
