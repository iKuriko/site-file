---
title: "Switch Port Mirroring"
date: 2026-03-10T17:38:19+08:00
draft: true
tags:
  - Other
description: 在交换机上将目标端口的流量复制到观察端口，以达到不影响业务的情况下也能进行数据包分析
---



在华为S6720交换机上抓取指定端口的报文，通常需要配置端口镜像（也称为端口镜像或SPAN）功能，将目标端口的流量复制到观察端口，再通过抓包工具（如Wireshark）在观察端口连接的设备上捕获数据。



## 配置观察端口（镜像目标端口）

将某个空闲端口（如GigabitEthernet 0/0/1）设置为观察端口，用于接收目标端口的镜像流量：

```
<HUAWEI> system-view

[HUAWEI] observe-port 1 interface GigabitEthernet 0/0/1
```

注释：

\- observe-port 1：定义观察端口的索引号为1。

\- interface GigabitEthernet 0/0/1：指定观察端口为G0/0/1。



## 配置镜像端口（需抓包的端口）

进入需要抓包的端口（如G0/0/2），将其流量镜像到观察端口：

```
[HUAWEI] interface GigabitEthernet 0/0/2

[HUAWEI-GigabitEthernet0/0/2] port-mirroring to observe-port 1 both
```

注释：

\- both：表示同时捕获入方向（inbound）和出方向（outbound）的流量。若只需单向，可替换为`inbound`或`outbound`。



## 验证镜像配置

查看当前端口镜像状态：

```
[HUAWEI] display port-mirroring
```

输出应包含镜像端口与观察端口的对应关系，例如：

```
Observe-port 1 : GigabitEthernet0/0/1
Port-mirror:
    Mirror-port               Direction  Observe-port             
1    GigabitEthernet0/0/2      Both       Observe-port 1
```



## 物理连接与抓包

1. 连接观察端口：将观察端口（G0/0/1）通过网线连接到抓包设备（如笔记本电脑）。
2. 启动抓包工具：在抓包设备上使用Wireshark等工具，选择对应网卡并开始捕获流量。



## 注意事项

1. 镜像方向限制：

   \- 华为交换机通常支持双向流量镜像（`both`），但某些型号可能仅支持入方向（如部分路由器仅支持入方向抓包）。

2. 性能影响：

   \- 镜像可能增加交换机负载，若目标端口流量过大，观察端口可能因速率限制导致丢包。

3. 配置保存：

   \- 镜像配置默认不保存，需通过`save`命令手动保存配置。

4. 光电复用端口：

   \- 若使用COMBO口（光电复用），需确认当前启用的端口类型（电口或光口）。



## 扩展：使用命令行直接抓包（部分型号支持）

部分华为设备支持通过命令行直接捕获报文（如路由器）：

```
<HUAWEI> capture-packet interface GigabitEthernet 0/0/2 destination terminal
```

\- 此命令将抓包结果直接输出到终端，但可能受限于缓冲区大小和性能。



## 总结

通过端口镜像功能，可以高效捕获指定端口的流量。若需长期监控，建议配置镜像规则后保存配置。实际操作中需注意交换机的型号限制和流量负载情况。更多细节可参考华为官方文档或相关配置指南。