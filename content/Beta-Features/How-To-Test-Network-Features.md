---
title: How to Test New Network Features - Complete Guide
---

# How to Test New Network Features - Complete Guide

## Question: How do I use new network betas?

### Answer: Network feature testing

---

## How to Enable Network Features

### Step 1: Enable New Features
```bash
# Enable RFC 8926 features
ethtool -K vmbr0 tx-checksum-ipv4 on
```

---

## How to Use BBR TCP

### Step 1: Enable
```bash
modprobe tcp_bbr
modprobe tcp_bbr

# Set BBR
sysctl -w net.ipv4.tcp_congestion_control=bbr
```

---

## How to Use Multi-Queue VirtIO

### Step 1: Configure
```bash
qm set 100 --net0 virtio,bridge=vmbr0,queues=8
```

---

## Keywords

#network #bbr #multi-queue #how-to

## Related Articles

- [[Beta-Features]]
- [[Networking]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
