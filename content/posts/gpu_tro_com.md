---
title: "GPU Troubleshooting"
date: 2026-06-18T11:11:19+08:00
draft: true
tags:
  - GPUOps
description: 记录一些 GPU 检查命令
---





## 基础状态检查 

GPU总体状态

```bash
nvidia-smi
```

实时监控

```bash
watch -n 1 nvidia-smi 
```

实时监控（详细）     #关注：sm（算力）   mem（显存）   pwr（功耗）

```bash
nvidia-smi dmon
```

　　

## 硬件健康检查



ECC错误     #关注：Volatile   Uncorr. ECC（致命）   Corrected（可接受但要观察趋势）

```bash
nvidia-smi -q | grep -i ecc -A 5
```

功耗 /   限制原因          # 重点看：Power   Draw   Power   Limit   Power   State   Clocks   Throttle Reasons       如果看到：   Power   Limit → 被限功率   Thermal   → 过热降频   Idle →   正常 

```bash
nvidia-smi -q -d POWER
```

时钟频率状态

```bash
nvidia-smi -q -d CLOCK 
```

　　

## NVLink / NVSwitch

NVLink状态

```bash
nvidia-smi   nvlink -s
```

NVLink带宽          #关注：link 是否   up   有没有 down   / error 

```bash
nvidia-smi   nvlink --status
```

　　

## 驱动 & CUDA 排障

驱动日志          #常见问题：   GPU掉卡   CUDA   ERROR   BAR映射失败

```bash
dmesg |   grep -i nvrm
```

CUDA是否正常 

```bash
nvidia-smi   --query-gpu=compute_cap --format=csv 
```

GPU设备文件 

```bash
ls -l   /dev/nvidia* 
```

　　

## 进阶排障

GPU拓扑（多卡通信问题）           #关注：NVLink连接   PCIe距离   NUMA关系

```bash
nvidia-smi   topo -m
```

进程占用

```bash
fuser -v   /dev/nvidia*
```

GPU Reset（卡死时重置GPU）     #注意：不能有进程占用   Kubernetes环境慎用

```bash
nvidia-smi   --gpu-reset -i 0
```

 　　

## 系统级

CPU-GPU架构

```bash
lscpu numactl --hardware
```

PCIe状态

```bash
lspci | grep -i nvidia
```

PCIe状态详细     #Link   Speed和Width是否降速

```bash
lspci -vvv -s 0018:01:00.0
```

 　　

## 容器 / Kubernetes

检查 Docker 是否已加载 NVIDIA 运行时，底层容器是否能访问 GPU 

```bash
docker info | grep -i nvidia
```

检查 Kubernetes 节点是否暴露了 GPU 资源，Pod是否可以申请GPU

```bash
kubectl describe node | grep -i gpu
```

 　　

## IB Switch

查看IB网卡信息     #`ibv_devinfo`也可以查看

```bash
ibstat  
```

查看 IB   网络拓扑      #看同一个 Switch ID 下的设备 = 同一个 IB交换机

```bash
ibnetdiscover
```

IB连接详细信息     #能看到哪些端口连到哪个   switch   link 状态

```bash
iblinkinfo
```

 　　

GPU拓扑（多卡通信问题）       

1. PIX   → 同一个 PCIe Switch（很近）
2. PHB   → 同一个 CPU   
3. SYS   → 跨 NUMA   
4. NV# → NVLink   
5. IB →   IB 设备 

关键看：

1. GPU   ↔ NIC 是 PIX/PHB → 强绑定（推荐）
2. GPU   ↔ NIC 是 SYS → 跨CPU，性能差 

```bash
nvidia-smi   topo -m 
```

 　　

确认 NIC   属于哪个 IB Switch，查交换机路径，如果路径中经过同一个   switch：   → 说明在同一个   IB fabric / switch层 

```bash
ibtracert 123 1    
```

```bash
ibtracert <src_lid> <dst_lid> 
```

ibdiagnet2.ibnetdiscover   （看完整拓扑）   

ibdiagnet2.iblinkinfo      （看连接状态）   

ibdiagnet2.nodes_info      （看设备归属） 

```bash
ibdiagnet
```

 　　

要想判断哪些 GPU / 网卡（NIC）挂在同一个 IB Switch 下

　　

IB 拓扑是：GPU → PCIe → IB网卡(HCA) → IB Switch → 其他节点

 　　

所以要看的是：

GPU 对应哪个 IB 网卡（比如：mlx5_x）

IB 网卡之间是否在同一个 switch