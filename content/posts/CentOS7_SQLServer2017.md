---
title: "CentOS7 安装 SQLServer2017"
date: 2021-11-18T11:56:06+08:00
draft: true
tags:
  - Linux
description: 
---



在CentoOS上安装sqlserver，需要先下载微软官方提供的在线yum源，再安装mssql-server软件包，再根据需要安装sqlserver的命令行工具或远程客户端进行连接

>OS必须条件  
>Memory：3.25 GB  
>File System：XFS or EXT4 (other file systems, such as BTRFS, are unsupported)  
>Disk space ：6 GB  
>Processor speed：2 GHz  
>Processor cores：2 cores  
>Processor type：x64-compatible only  

官方文档：<https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-red-hat> 

## 安装mssql-server

下载阿里云在线yum源到本地

```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

下载微软官方的sqlserver源到本地

```bash
wget -O /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/7/mssql-server-2017.repo
```

最好查看一下内容是否正确

```bash
cat /etc/yum.repos.d/mssql-server.repo
```

安装mssql-server（SQL Server软件包）

```bash
yum install -y mssql-server
```

运行mssql-conf配置文件，选择SQL版本并配置SA密码（密码必须符合策略要求 ）

```bash
/opt/mssql/bin/mssql-conf setup
```

这里有个小问题，服务器内存小于2G是无法正常安装的，所以需要调整内存或者自行搜索解决



初始化完成后，查看服务运行状态

```bash
systemctl status mssql-server
```



要允许远程连接，允许firewalld防火墙上的SQLServer端口。 默认的 SQL Server 端口为 TCP 1433。

```bash
sudo firewall-cmd --zone=public --add-port=1433/tcp --permanent
sudo firewall-cmd --reload
```



## 安装sqlserver命令行工具

>这时候安装已经完成，但是若要创建数据库或者本地连接数据库，则需要使用一个能够在SQLServer上运行Transact-SQL语句的工具进行连接。 以下步骤安装 SQL Server 命令行工具：sqlcmd和bcp。 

下载微软官方的软件包yum源

```bash
wget -O  /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/7/prod.repo
```

如果以前装过mssql，则需要删除较旧的UnixODBC软件包

```bash
yum remove unixODBC-utf16 unixODBC-utf16-devel 
```

安装mssql工具包和UnixODBC开发人员软件包

```bash
yum install -y mssql-tools unixODBC-devel 
```

为了方便使用工具包中的命令，添加PATH环境变量

```bash
echo "export PATH=$PATH:/opt/mssql-tools/bin" >> /etc/profile
source /etc/profile
```

使用sqlcmd命令连接本地的sqlserver，输入之前设置的SA密码

```bash
sqlcmd -S localhost -U SA -p
```

如果出现以下报错

```bash
Sqlcmd: Error: Microsoft ODBC Driver 17 for SQL Server : Login timeout expired.
Sqlcmd: Error: Microsoft ODBC Driver 17 for SQL Server : TCP Provider: Timeout error [258]. .
Sqlcmd: Error: Microsoft ODBC Driver 17 for SQL Server : Unable to complete login process due to delay in prelogin response.
```

将localhost换成127.0.0.1，再次连接

```bash
sqlcmd -S 127.0.0.1 -U SA -p
```

除了 sqlcmd, bcp, SSMS (on Windows)，还可以使用以下工具：

- SQL Operations Studio
- mssql-cli
- Visual Studio Code
- Navicat客户端远程连接



## 检查安装的SQL Server版本

```bash
sqlcmd -S localhost -U SA -Q 'select @@VERSION'
```



## 卸载 SQL Server

>如果安装了错误的版本或者需要卸载

只移除软件包本身

```bash
yum remove mssql-server
```

移除包并不会删除生成的数据库文件。 需要手动删除

```bash
rm -rf /var/opt/mssql/
```





