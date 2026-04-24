---
title: Proxmox VE Beta Features Guide
---

# Proxmox VE Beta Features Guide

## Table of Contents

1. [Experimental Features](#experimental-features)
2. [v4 Features](#v4-features)
3. [Upcoming Features](#upcoming-features)
4. [Feature Gates](#feature-gates)
5. [Testing Features](#testing-features)
6. [Feature Comparison](#feature-comparison)

---

## Experimental Features

### Enable Experimental Features

```bash
# In /etc/pve.config
experimental: 1
```

### Memory Ballooning

```bash
# Enable balloon
qm set <vmid> --balloon 2048

# Check balloon status
qm config <vmid> | grep balloon
```

### CPU Hotplug

```bash
# Enable CPU hotplug
qm set <vmid> --cpu hotplug

# Add CPU without restart
qm set <vmid> --cores 4
```

### Memory Hotplug

```bash
# Enable memory hotplug
qm set <vmid> --memory hotplug

# Resize without restart
qm set <vmid> --memory 8192
```

---

## v4 Features

### ZFS v4 Features

```bash
# ZFS fast dedup
zfs set dedup=fast dedup-pool

# ZFS zstd compression
zfs set compression=zstd pool
zfs get compression pool
```

### New Storage Features

```bash
# pvesm thin provisioning
pvesm add lvmthin pool --thin-pool

# Storage replication
pvesm replicate storage --source local --target remote
```

---

## Upcoming Features

### v5 Features Preview

| Feature | Status | Description |
|---------|--------|-------------|
| PCIe 5.0 | Beta | 32 GT/s transfer |
| NVMe-oF | Beta | NVMe over Fabrics |
| Remote Blob | Beta | Remote storage blob |
| TPM 2.0 | Stable | Trusted Platform Module |

### Testing New Features

```bash
# Enable beta repos
echo "deb http://download.proxmox.com/debian/pve bookworm pvetest" > /etc/apt/sources.list.d/pvetest.list
apt update && apt upgrade -y
```

---

## Feature Gates

### Enable Feature Gates

```bash
# /etc/modprobe.d/kvm.conf
options kvm_intel nested=1 ept=1

# Runtime enable
modprobe kvm_intel nested=1
```

### Check Feature Support

```bash
# CPU features
cat /proc/cpuinfo | grep flags | head -1

# KVM features
ls -la /sys/module/kvm*/parameters/

# SVM features
cat /sys/module/kvm_amd/parameters
```

---

## Testing Features

### Benchmark Tools

```bash
# Install bench
apt install -y bonnie++ fio sysbench

# Quick test
fio --name=randwrite --ioengine=libaio --bs=4k --direct=1 --size=1G --numjobs=4 --runtime=30

# CPU bench
sysbench cpu --cpu-max-prime=20000 run

# Memory bench
sysbench memory --memory-block-size=1M --memory-total-size=10G run
```

---

## Feature Comparison

### Performance Comparison

| Feature | Proxmox 7 | Proxmox 8 |
|--------|-----------|-----------|
| ZFS | v2 | v3 |
| Ceph | Quincy | Reef |
| QEMU | 7.0 | 8.0 |
| Kernel | 5.15 | 6.2 |
| Storage | LVM-Thin | LVM-Thin v2 |

---

## Keywords

#beta #experimental #features #new #upcoming

## Links

- [[Proxmox-VE]] - Main documentation
- [[Advanced]] - Advanced configuration
---

[[index|Back to Proxmox VE]]
