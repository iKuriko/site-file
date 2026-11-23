---
title: "Ntopng 流量监控"
date: 2026-07-28T11:31:57+08:00
draft: true
tags:
  - Networking
description: 在 CentOS8 上部署 ntopng ，实现性能网络流量监控与底层调优
---

# Ntopng 高性能网络流量监控与底层调优实战

> **环境元数据**
>
> - **监控网卡**：`enp3s0`
> - **本地监控网段**：`192.168.12.0/24`
> - **Web 控制台**：`[http://192.168.12.137:3000](http://192.168.12.137:3000)`
> - **加速抓包框架**：PF_RING + 网卡 RSS 多队列 + CPU 中断绑定

---

## 1. 基础环境与 ntopng 安装

安装底层依赖、Redis 及 MaxMind GeoIP 支持：

```bash
dnf install -y wget curl net-tools gcc gcc-c++ make \
    kernel-devel-$(uname -r) kernel-headers-$(uname -r) \
    libpcap libpcap-devel redis hiredis hiredis-devel \
    libmaxminddb libmaxminddb-devel
```

配置官方仓库并安装 ntopng：

```bash
wget https://packages.ntop.org/centos-stable/ntop.repo -O /etc/yum.repos.d/ntop.repo
yum install -y ntopng
ntopng --version
```

配置基础运行参数：

```bash
cat > /etc/ntopng/ntopng.conf << 'EOF'
-G=/var/run/ntopng.pid
-i=enp3s0
-w=3000
-m=192.168.12.0/24
-n=1
-d=/var/lib/ntopng
-q
EOF
```

启动并验证服务：

```bash
systemctl enable --now ntopng
systemctl status ntopng
journalctl -u ntopng -f
```

---

## 2. 集成 PF_RING 高性能抓包模块

在大流量场景下，默认的 libpcap 容易出现严重丢包，需集成 PF_RING 绕过部分协议栈开销：

```bash
# 从官方仓库安装
dnf install -y pfring-dkms pfring

# 手动加载内核模块
modprobe pf_ring

# 验证模块加载
lsmod | grep pf_ring
cat /proc/net/pf_ring/info

# 配置开机自动加载
echo "pf_ring" > /etc/modules-load.d/pf_ring.conf

# 调整 ring buffer 槽位数
cat > /etc/modprobe.d/pf_ring.conf << 'EOF'
options pf_ring min_num_slots=65536 transparent_mode=0
EOF

# 检查 ntopng 是否识别 PF_RING
ntopng -h 2>&1 | grep -i "pfring\|PF_RING"
```

---

## 3. 网卡 RSS 多队列与中断调优

充分利用多核 CPU 分散抓包压力，避免单核软中断（softirq）跑满导致丢包。

### 3.1 开启网卡多队列与 RSS 哈希

```bash
# 查看网卡当前与最大支持队列数
ethtool -l enp3s0

# 开启多队列（结合 CPU 核数调整，此处设为 4 队列）
ethtool -L enp3s0 combined 4

# 配置五元组哈希分流
ethtool -N enp3s0 rx-flow-hash tcp4 sdfn
ethtool -N enp3s0 rx-flow-hash udp4 sdfn
ethtool -N enp3s0 rx-flow-hash tcp6 sdfn
ethtool -N enp3s0 rx-flow-hash udp6 sdfn

# 增大网卡 Ring Buffer 至最大值
ethtool -G enp3s0 rx 4096 tx 4096
```

### 3.2 停用 irqbalance 并手动绑定中断 CPU 亲和性

```bash
systemctl stop irqbalance
systemctl disable irqbalance

# 查看网卡中断号分布
grep enp3s0 /proc/interrupts

# 依据中断号将每个队列绑定到独立 CPU 核心（示例中断号 28-31 绑定至 CPU 0-3）
echo 1 > /proc/irq/28/smp_affinity   # CPU0
echo 2 > /proc/irq/29/smp_affinity   # CPU1
echo 4 > /proc/irq/30/smp_affinity   # CPU2
echo 8 > /proc/irq/31/smp_affinity   # CPU3
```

### 3.3 网卡参数持久化脚本与 Systemd 服务

```bash
cat > /usr/local/bin/nic-tuning.sh << 'EOF'
#!/bin/bash
NIC="enp3s0"

# 多队列与 Ring Buffer
ethtool -L $NIC combined 4
ethtool -G $NIC rx 4096 tx 4096

# RSS 五元组哈希
ethtool -N $NIC rx-flow-hash tcp4 sdfn
ethtool -N $NIC rx-flow-hash udp4 sdfn

# 关闭 Offload 特性（避免大包聚合影响抓包准确性）
ethtool -K $NIC gro off
ethtool -K $NIC lro off
ethtool -K $NIC tso off
ethtool -K $NIC gso off
ethtool -K $NIC rx-gro-hw off

# 停止动态调优服务
systemctl stop irqbalance
EOF

chmod +x /usr/local/bin/nic-tuning.sh

cat > /etc/systemd/system/nic-tuning.service << 'EOF'
[Unit]
Description=NIC Performance Tuning for ntopng
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/nic-tuning.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable nic-tuning
```

---

## 4. 系统内核参数与 Limits 优化

```bash
cat >> /etc/sysctl.conf << 'EOF'
# ===== ntopng 大流量抓包优化 =====
net.core.rmem_max = 134217728
net.core.rmem_default = 16777216
net.core.wmem_max = 134217728
net.core.wmem_default = 16777216
net.core.netdev_max_backlog = 300000
net.core.netdev_budget = 600
net.core.netdev_budget_usecs = 10000
net.core.somaxconn = 65535
fs.file-max = 1000000
vm.swappiness = 10
vm.dirty_ratio = 40
vm.dirty_background_ratio = 10
EOF

sysctl -p

# 增大文件描述符上限
cat >> /etc/security/limits.conf << 'EOF'
*    soft    nofile    1000000
*    hard    nofile    1000000
root soft    nofile    1000000
root hard    nofile    1000000
EOF
```

---

## 5. 配置 ntopng 启用 PF_RING 并重启

更新 `/etc/ntopng/ntopng.conf`：

```bash
cat > /etc/ntopng/ntopng.conf << 'EOF'
# 使用 PF_RING 驱动捕获流量
-i=pfring:enp3s0

# 基础进程与网络配置
-G=/var/run/ntopng.pid
-m=192.168.12.0/24
-n=1
-q

# 大流量与高并发连接池优化
--max-num-hosts=262144
--flow-timeout=300
--tcp-flow-timeout=3600
EOF

systemctl restart ntopng
journalctl -u ntopng -f | grep -i "pfring\|PF_RING\|interface"
```

---

## 6. 性能指标校验与丢包监控

```bash
# 查看 PF_RING 统计
cat /proc/net/pf_ring/info

# 检查网卡硬件层丢包
ethtool -S enp3s0 | grep -i drop
ip -s link show enp3s0

# 持续观察网卡丢弃与错包指标
watch -n 2 'ethtool -S enp3s0 | grep -i "rx_dropped\|rx_missed"'
```

### 抓包方案吞吐与丢包性能对比

| 捕获框架 | 单核吞吐能力 | 丢包表现 |
| :--- | :--- | :--- |
| **libpcap（默认）** | ~400-600 Mbps | 高并发、大流量时丢包严重 |
| **PF_RING（单队列）** | ~1-2 Gbps | 偶发轻微丢包 |
| **PF_RING + RSS 4队列** | ~8-10 Gbps | 吞吐高，基本无丢包 |
| **PF_RING ZC (Zero Copy)** | 40-100 Gbps | 零拷贝极致性能，零丢包 |

---

## 7. 日志管理与 Logrotate 轮转配置

### 7.1 配置 logrotate

```bash
cat > /etc/logrotate.d/ntopng << 'EOF'
/var/log/ntopng/*.log {
    daily
    rotate 30
    maxsize 500M
    compress
    delaycompress
    missingok
    notifempty
    create 0644 ntopng ntopng
    sharedscripts
    postrotate
        /bin/kill -HUP $(cat /var/run/ntopng.pid 2>/dev/null) 2>/dev/null || true
        /bin/systemctl reload ntopng 2>/dev/null || true
    endscript
}
EOF

# 测试配置（Dry-run 调试）
logrotate -d /etc/logrotate.d/ntopng

# 强制触发一次测试
logrotate -f /etc/logrotate.d/ntopng
```

### 7.2 Systemd Journald 日志容量约束（若未输出文件日志）

```bash
cat >> /etc/systemd/journald.conf << 'EOF'
[Journal]
Storage=persistent
SystemMaxUse=2G
SystemMaxFileSize=200M
MaxRetentionSec=30day
SyncIntervalSec=6h
EOF

systemctl restart systemd-journald

# 手动清理超量日志示例
journalctl --vacuum-time=7d
journalctl --vacuum-size=1G
```