---
title: "Gpu_tro_com"
date: 2026-06-18T11:11:19+08:00
draft: true
tags:
  - GPUOps
description: 记录一些 GPU 检查命令
---







基础状态检查

 

| nvidia-smi              | GPU总体状态                                                  |
| ----------------------- | ------------------------------------------------------------ |
| watch -n 1   nvidia-smi | 实时监控                                                     |
| nvidia-smi dmon         | 实时监控（详细）   关注：      sm（算力）   mem（显存）   pwr（功耗） |

 

 

硬件健康检查

| nvidia-smi   -q \| grep -i ecc -A 5 | ECC错误          关注：   Volatile   Uncorr. ECC（致命）   Corrected（可接受但要观察趋势） |
| ----------------------------------- | ------------------------------------------------------------ |
| nvidia-smi   -q -d POWER            | 功耗 /   限制原因          重点看：   Power   Draw   Power   Limit   Power   State   Clocks   Throttle Reasons       如果看到：   Power   Limit → 被限功率   Thermal   → 过热降频   Idle →   正常 |
| nvidia-smi   -q -d CLOCK            | 时钟频率状态                                                 |

 

 

NVLink / NVSwitch

| nvidia-smi   nvlink -s       | NVLink状态                                                   |
| ---------------------------- | ------------------------------------------------------------ |
| nvidia-smi   nvlink --status | NVLink带宽          关注：   link 是否   up   有没有 down   / error |

 

 

驱动 & CUDA 排障

| dmesg \|   grep -i nvrm                           | 驱动日志          常见问题：   GPU掉卡   CUDA   ERROR   BAR映射失败 |
| ------------------------------------------------- | ------------------------------------------------------------ |
| nvidia-smi   --query-gpu=compute_cap --format=csv | CUDA是否正常                                                 |
| ls -l   /dev/nvidia*                              | GPU设备文件                                                  |

 

 

进阶排障

| nvidia-smi   topo -m          | GPU拓扑（多卡通信问题）            关注：   NVLink连接   PCIe距离   NUMA关系（GB200   + Grace 很关键） |
| ----------------------------- | ------------------------------------------------------------ |
| fuser -v   /dev/nvidia*       | 进程占用                                                     |
| nvidia-smi   --gpu-reset -i 0 | GPU Reset（卡死时重置GPU）          注意：   不能有进程占用   Kubernetes环境慎用 |

 

 

系统级

| lscpu      numactl --hardware | CPU-GPU架构（GB200   = Grace + GPU）                         |
| ----------------------------- | ------------------------------------------------------------ |
| lspci \|   grep -i nvidia     | PCIe状态                                                     |
| lspci   -vvv -s 0018:01:00.0  | PCIe状态详细          关注：   Link   Speed（是否降速）   Width（x16   是否变 x8） |





IB Switch

想要判断哪些 GPU / 网卡（NIC）挂在同一个 IB Switch 下

 

IB 拓扑是：

 

GPU → PCIe → IB网卡(HCA) → IB Switch → 其他节点

 

所以要看的是：

GPU 对应哪个 IB 网卡（mlx5_x）

IB 网卡之间是否在同一个 switch

 

ib网卡检查

| Ibstat   或   ibv_devinfo                                  | 查看IB网卡信息                                               |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| ibnetdiscover                                              | 查看 IB   网络拓扑          关注：      看同一个 Switch ID 下的设备 = 同一个 IB交换机 |
| iblinkinfo                                                 | IB连接详细信息          可以看到：   哪些端口连到哪个   switch   link 状态 |
| nvidia-smi   topo -m                                       | GPU拓扑（多卡通信问题）       解释：   PIX   → 同一个 PCIe Switch（很近）   PHB   → 同一个 CPU   SYS   → 跨 NUMA   NV# → NVLink   IB →   IB 设备       关键：   GPU   ↔ NIC 是 PIX/PHB → 强绑定（推荐）   GPU   ↔ NIC 是 SYS → 跨CPU，性能差 |
| ibtracert   123 1            ibtracert <src_lid> <dst_lid> | 确认 NIC   属于哪个 IB Switch，查交换机路径          如果路径中经过同一个   switch：   → 说明在同一个   IB fabric / switch层 |
| ibdiagnet                                                  | ibdiagnet2.ibnetdiscover   （看完整拓扑）   ibdiagnet2.iblinkinfo      （看连接状态）   ibdiagnet2.nodes_info      （看设备归属） |