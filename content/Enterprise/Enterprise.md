# Proxmox VE Enterprise Guide

## Table of Contents

1. [Enterprise Features](#enterprise-features)
2. [Enterprise Support](#enterprise-support)
3. [High Availability](#high-availability)
4. [Enterprise Storage](#enterprise-storage)
5. [Backup Solutions](#backup-solutions)

---

## Enterprise Features

### Subscription

| Feature | Community | Community+ | Premier |
|---------|-----------|------------|----------|
| Updates | No | Yes | Yes |
| Support | Community | Email | 24/7 |
| Security | No | Yes | Yes |

### Enable Subscription

```bash
# Add enterprise repo
echo "deb http://download.proxmox.com/debian/pve bookworm pve-enterprise" > /etc/apt/sources.list.d/pve-enterprise.list
apt update
```

---

## Enterprise Support

### Getting Support

- **Email**: support@proxmox.com
- **Forum**: forum.proxmox.com
- **Documentation**: pve.proxmox.com/wiki/

---

## High Availability

### HA Setup

```bash
# Enable HA
ha-manager enable <vmid>

# Configure HA group
ha-manager group add production pve1 pve2 pve3

# Set VM priority
ha-manager crm add production --vm <vmid> --priority 100
```

---

## Enterprise Storage

### Ceph Enterprise

```bash
# Install Ceph
pveceph install

# Create cluster
pveceph cluster create

# Add OSDs
pveceph osd create /dev/sda
pveceph osd create /dev/sdb
```

### ZED Enterprise

```bash
# ZFS Enterprise features
zpool create -f mirror pool mirror /dev/sda /dev/sdb
zfs set compression=zstd pool
zfs set checksum=sha256 pool
```

---

## Backup Solutions

### Proxmox Backup Server

```bash
# Install PBS
apt install proxmox-backup-server

# Configure
# Web UI: https://192.168.1.100:8007
```

### Enterprise Backup

```bash
# Schedule backup
# Datacenter → Backup → Add
# Storage: PBS
# Schedule: Daily
# Compression: zstd
# Retention: 7 daily, 4 weekly
```

---

## Keywords

#enterprise #business #ha #support

## Links

- [[Proxmox-VE]] - Main documentation
- [[Cluster]] - Clustering
- [[Backup]] - Backup