---
title: Ceph
---

# Ceph

## Overview

Ceph is a distributed storage system that provides object, block, and file storage in a single unified system.

## Requirements

| Component | Minimum |
|------------|----------|
| OSDs | 3 |
| MONs | 3 |
| RAM | 4 GB per OSD |
| Disk | Separate SSD for journal |

## Install Ceph

```bash
apt install ceph -y
```

## Create Pool

```bash
ceph osd pool create pool1 64 64
ceph osd pool set pool1 pg_num 128
ceph osd pool set pool1 pgp_num 128
```

## Commands

| Command | Description |
|---------|-------------|
| `ceph -s` | Show status |
| `ceph osd ls` | List OSDs |
| `ceph osd pool ls` | List pools |
| `ceph health` | Check health |

## See Also

- [[Storage]]
- [[LVM-Management]]
- [[ZFS-Tuning]]
[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
