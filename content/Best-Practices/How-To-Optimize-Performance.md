# How to Optimize Performance - Complete Guide

## Question: How do I optimize Proxmox performance?

### Answer: Apply these tuning settings

---

## How to Optimize CPU

### Step 1: Enable Performance Governor
```bash
# Set all cores to performance
for cpu in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor; do
    echo performance > $cpu
done

# Make persistent
echo 'performance' > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```

---

## How to Optimize Memory

### Step 1: Tune sysctl
```bash
cat >> /etc/sysctl.conf << EOF
vm.swappiness=10
vm.min_free_kbytes=65536
vm.vfs_cache_pressure=50
EOF

sysctl -p
```

---

## How to Optimize Network

### Step 1: Network Tuning
```bash
cat >> /etc/sysctl.conf << EOF
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 87380 16777216
net.ipv4.tcp_congestion_control=bbr
EOF

sysctl -p
```

---

## How to Optimize Disk I/O

### Step 1: Set I/O Scheduler
```bash
# For SSD/NVMe
echo none > /sys/block/sda/queue/scheduler

# For HDD
echo deadline > /sys/block/sda/queue/scheduler

# Make persistent
cat >> /etc/rc.local << EOF
echo none > /sys/block/sda/queue/scheduler
EOF
```

---

## Keywords

#performance #optimization #cpu #memory #network #disk #how-to

## Related Articles

- [[Best-Practices]]
- [[Performance]]
- [[Proxmox-VE]]