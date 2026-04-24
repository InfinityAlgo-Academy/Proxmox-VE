# Ceph Storage Configuration

## Overview

Ceph integrated in Proxmox VE provides a unified, scalable software-defined storage solution for block, object, and file storage.

## Install Ceph

```bash
# Install Ceph
pveceph install

# Or via web UI:
# Datacenter -> Storage -> Add -> Ceph
```

### Setup Steps

1. **Install Ceph packages**
2. **Create monitor**
3. **Create OSDs**
4. **Create pools**

## Ceph Components

### MON (Monitor)

```bash
# Create monitor
pveceph mon create

# Check status
pveceph mon status
```

### OSD (Object Storage Daemon)

```bash
# Create OSD
pveceph osd create /dev/sdb

# Create OSD with journal
pveceph osd create /dev/sdc --journal /dev/sdd
```

### MDS (Metadata Server)

```bash
# Create MDS
pveceph mds create

# Verify
pveceph mds status
```

## Ceph Pools

### Create Pool

```bash
# Create pool
pveceph pool create vm-data

# Configure pool
ceph pool set vm-data size 3
ceph pool set vm-data min_size 2
ceph pool set vm-data pg_num 128
```

### Pool Configuration

| Setting | Description | Default |
|---------|-------------|---------|
| size | Replicas | 3 |
| min_size | Min replicas | 2 |
| pg_num | Placement groups | 128 |
| crush_rule | Rule to use | - |

### Pool Properties

```bash
# Set replicas
ceph pool set vm-data size 3

# Enable compression
ceph pool set vm-data compression_algorithm=zstd

# Set min size
ceph pool set vm-data min_size 2
```

## Ceph CRUSH Map

### Custom Rules

```bash
# Create rule
ceph osd crush rule create-replicated replicated-rule default host

# Apply rule to pool
ceph osd pool set vm-data crush_rule replicated-rule
```

### Tunables

```bash
# View current
ceph osd crush dump

# Adjust
ceph osd crush tunables legacy
```

## Ceph Performance

### Tuning OSD

```bash
# Increase thread count
ceph config set osd osd_max_thread_ops 5000
ceph config set osd osd_op_threads 16
```

### Cache Tiering

```bash
# Create cache tier
ceph tier add vm-data-cache vm-data --force

# Configure cache
ceph tier cache-mode vm-data-cache writeback

# Set target
ceph tier set-target vm-data-cache .5
```

## Ceph Backup/Restore

### Backup

```bash
# RBD snapshot
rbd snap create vm-data/vm-100@snapshot-$(date +%Y%m%d)

# Export
rbd export vm-data/vm-100@snapshot-$(date +%Y%m%d) /backup/vm-100.img
```

### Restore

```bash
# Import
rbd import /backup/vm-100.img vm-data/vm-100-restored

# Re-size if needed
rbd resize vm-data/vm-100-restored --size 100G
```

## Troubleshooting Ceph

### Check Health

```bash
# Ceph health
ceph health

# Detailed
ceph -s

# PG status
ceph pg stat
```

### Common Issues

```bash
# OSD down
ceph osd out 0
ceph osd in 0

# PG stuck
ceph pg dump
ceph health detail
```

## Keywords

#ceph #osd #mon #mds #pool #crush #replication #software-defined

## Related Articles

- [[Advanced]]
- [[Storage]]
- [[ZFS-Tuning]]