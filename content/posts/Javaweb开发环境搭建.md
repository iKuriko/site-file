---
title: "Javaweb开发环境搭建"
date: 2021-09-09T16:53:45+08:00
draft: true
---
为 Javaweb 开发项目搭建基础环境，全部使用源码包手动安装

环境使用软件版本为：Centos7.9_2009 + jdk1.8 + mysql 5.7.34 + python3.6 + tomcat 8.5.69

## Oracle jdk1.8.0_301
#软件包下载地址： https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html

解压软件包到opt目录下

```bash
tar -zxvf jdk-8u301-linux-x64.tar.gz -C /opt/
```

创建软连接以便版本更迭

```bash
ln -s /opt/jdk1.8.0_301/ /opt/jdk
```

添加系统环境变量，立即生效

```bash
vim /etc/profile
#Set the JDK environment variables
export JAVA_HOME=/opt/jdk
export JRE_HOME=/opt/jdk/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```
```bash
source /etc/profile
```

查看java版本，是否生效

```bash
java -version
```



## MySQL 5.7.34

#MySQL软件包下载地址： https://downloads.mysql.com/archives/community/

检查并删除自带的mariadb

```bash
yum list installed  | grep mariadb
yum -y remove mariadb-libs.x86_64
```

解压软件包到/opt下

```bash
tar -zxvf mysql-5.7.34-el7-x86_64.tar.gz -C /opt
```

创建软连接

```bash
ln -s /opt/mysql-5.7.34-el7-x86_64.tar.gz /opt/mysql
```

创建用户，-M不指定家目录，-s /sbin/nologin无法登录系统

```bash
useradd -M -s /sbin/nologin mysql
```

创建mysql数据目录，赋予mysql用户权限

```bash
mkdir -pv /data/mysql
chown -R mysql:mysql /data/mysql/
```

创建my.cnf配置文件

```bash
vim /etc/my.cnf
[mysqld]
bind-address=0.0.0.0
port=3306
user=mysql
basedir=/opt/mysql
datadir=/data/mysql
socket=/tmp/mysql.sock
log-error=/data/mysql/mysql.err
pid-file=/data/mysql/mysql.pid
#character config
character_set_server=utf8mb4
symbolic-links=0
```

将命令目录加入系统环境变量，立即生效

```bash
echo "PATH=$PATH:/opt/mysql/bin" >> /etc/profile
source /etc/profile
```

初始化数据库，指定配置文件，安装目录，数据目录，指定用户

```bash
mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/data/mysql/ --user=mysql --initialize
```

查看mysql日志，查看数据库初始密码

```bash
cat /data/mysql/mysql.err
2021-08-23T14:47:03.946617Z 1 [Note] A temporary password is generated for root@localhost: R4LagOyP26%,
```

拷贝启动脚本

```bash
cp /opt/mysql/support-files/mysql.server /etc/init.d/mysql
```

启动服务，设置开启自启

```bash
service mysql start
chkconfig mysql on
```

mysql默认监听3306端口

```bash
ss -utpln | grep 3306
tcp    LISTEN     0      80        *:3306                  *:*                   users:(("mysqld",pid=3824,fd=20))
```

使用刚才看到的密码，登录数据库，重置密码为root

```sql
mysql -u root -p
Enter password:
mysql>
SET PASSWORD = PASSWORD('root');
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
FLUSH PRIVILEGES;
```

mysql默认只能本地访问，需要修改下user表root用户的权限，将默认的localhost改为%，允许所有主机

```sql
use mysql;
update user set host = '%' where user = 'root';
flush privileges;
```

## Python 3.6

下载地址： https://www.python.org/ftp/python/3.6.2/Python-3.6.2.tgz

```bash
#可选的软件包

#yum install gcc patch libffi-devel python-devel  zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel -y

#gcc :c编译器
#python-devel    python 开发包
#openssl-devel     用于python的ssl模块
#sqlite-devel     轻量级数据库
```

安装gcc编译器
```bash
yum -y install gcc
```

解压软件包到/opt目录
```bash
tar -zxvf Python-3.6.2.tgz -C /opt/
```
创建软连接
```bash
ln -s /opt/Python-3.6.2/ /opt/python
```

添加环境变量，立即生效
```bash
vim /etc/profile
#Set the Python environment variables
export PATH=$PATH:/opt/python/bin
```
```bash
source /etc/profile
```

```
[root@test1 ~]# python3
Python 3.6.2 (default, Aug 24 2021, 20:05:45)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-44)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
>>>
>>>
```


## Tomcat 8.5.69

#下载地址： https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.69/

解压软件包到/opt目录

```bash
tar -zxvf apache-tomcat-8.5.69.tar.gz -C /opt
```

创建软连接

```bash
ln -s /opt/apache-tomcat-8.5.69/ /opt/tomcat
```

添加环境变量

```bash
#vim /opt/tomcat/bin/setclasspath.sh
#export JAVA_HOME=/opt/jdk
#export JRE_HOME=/opt/jdk/jre
#source setclasspath.sh
#已添加环境变量则无需再次添加
```

启动tomcat服务

```bash
cd /opt/tomcat/bin
./startup.sh
Using CATALINA_BASE:   /opt/tomcat
Using CATALINA_HOME:   /opt/tomcat
Using CATALINA_TMPDIR: /opt/tomcat/temp
Using JRE_HOME:        /opt/jdk/jre
Using CLASSPATH:       /opt/tomcat/bin/bootstrap.jar:/opt/tomcat/bin/tomcat-juli.jar
Using CATALINA_OPTS:   
Tomcat started.
```
开机自启动
```bash
vim /etc/rc.local
/bin/sh /opt/tomcat/bin/startup.sh &
```

tomcat默认监听8080端口
```bash
ss -utpln | grep 8080
tcp    LISTEN     0      100    [::]:8080               [::]:*                   users:(("java",pid=17268,fd=55))
```

