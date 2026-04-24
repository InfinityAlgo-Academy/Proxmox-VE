---
title: Complete Cluster Guide
tags:
  - cluster
  - ha
  - corosync
  - quorum
  - guide
---

# Complete Cluster Guide

## Overview

Proxmox VE clustering provides high availability, live migration, and centralized management.

## Requirements

| Component | Minimum | Recommended |
|-----------|---------|------------|
| Nodes | 3 | 3+ |
| Network | 1 Gbps | 10 Gbps |
| Same version | Yes | Yes |
| Storage | Shared | Shared |

## Create Cluster

### On First Node

```bash
# Create cluster
pvecm create production-cluster

# Verify
pvecm status
```

### Add Second Node

```bash
# From node2
pvecm add 192.168.1.101

# Or with SSH port
pvecm add 192.168.1.101 -port 22
```

### Add Third Node

```bash
pvecm add 192.168.1.102
```

## Cluster Management

### Show Status

```bash
# Cluster status
pvecm status

# Nodes
pvecm nodes

# Expected votes
pvecm expected
```

### Cluster Commands

| Command | Description |
|---------|-------------|
| `pvecm status` | Show cluster status |
| `pvecm nodes` | List nodes |
| `pvecm add ip` | Add node |
| `pvecm delnode name` | Remove node |
| `pvecm expected N` | Set quorum |

## Quorum

### Understanding Quorum

```
Nodes = 3
Quorum needed = (N/2)+1 = 2

Nodes = 2 (splits)
Quorum = lost (need 3)

Nodes = 3
Quorum = 2 (OK)
```

### Fix Quorum Issues

```bash
# Set expected votes (2 node cluster)
pvecm expected 2
```

## High Availability

### Enable HA

```bash
# Install
apt install ha-manager -y

# Create HA group
ha-manager group add production pve1 pve2 pve3
```

### Configure HA

```bash
# Enable HA for VM
ha-manager enable vm:100 --group production

# Enable HA for container
ha-manager enable ct:200 --group production

# Disable HA
ha-manager disable vm:100
```

### HA Commands

```bash
# HA status
ha-manager status

# Start VM with HA
ha-manager start vm:100

# Stop VM with HA  
ha-manager stop vm:100
```

## Live Migration

### Migrate VM

```bash
# Live migrate
qm migrate 100 node2 --online

# Offline migrate
qm migrate 100 node2 --offline
```

### Migrate Container

```bash
pct migrate 200 node2 --online
```

## Storage Sharing

### NFS for Cluster

```bash
# Add shared storage
pvesm add nfs shared-store \
  --server 192.168.1.200 \
  --export /mnt/shared \
  --content rootdir,images
```

### Ceph for Cluster

```bash
# Install Ceph
apt install ceph -y

# Create pool
ceph osd pool create pool1 64 64

# Add to Proxmox
pvesm add cephpool --poolname pool1
```

## Multi-Cluster

### Backup Cluster

```bash
# Create backup cluster
pvecm create backup-cluster
```

### Migrate Between Clusters

```bash
# Export
qm migrate 100 backup-node1 --offline

# Import (on backup cluster)
qm restore /backup/vm-100.tar.gz 100
```

## Troubleshooting

### Cluster Issues

| Issue | Solution |
|-------|---------|
| No quorum | Add votes, restart corosync |
| Node can't connect | Check network |
| Can't migrate | Check shared storage |

### Commands

```bash
# Restart cluster
systemctl restart corosync
systemctl restart pve-ha-lrm

# Show logs
journalctl -u pve-ha-lrm -f
```

## See Also

- [[index|Back to Proxmox VE]]
- [[High-Availability]]
- [[Cluster/How-To-Configure-Cluster]]
- [[Storage]]
- [[Migration]]