# Live Migration Advanced Guide

## Overview

Live migration moves running VMs between nodes without downtime. Essential for cluster maintenance, load balancing, and disaster recovery.

## Prerequisites

### Network Requirements

```bash
# High-speed network for migration
# Minimum 10Gbps recommended
# Same subnet or routed
```

### Storage Requirements

```bash
# Shared storage required:
# - NFS
# - iSCSI
# - Ceph
# - SMB
```

## Basic Live Migration

### Standard Migration

```bash
# Migrate VM to node2
qm migrate 100 pve2 --online

# Check status
qm status 100
```

### Migration with Storage

```bash
# Migrate to different storage
qm migrate 100 pve2 --storage local-fast
```

## Migration Options

### Bandwidth Limit

```bash
# Limit to 100Mbps
qm migrate 100 pve2 --bandwidth 100M

# Unlimited
qm migrate 100 pve2 --bandwidth 10G
```

### Compression

```bash
# Enable compression
qm migrate 100 pve2 --compression zstd

# Disable
qm migrate 100 pve2 --compression 0
```

### Migration Type

```bash
# Force stop/start (offline)
qm migrate 100 pve2 --force

# Live (default)
qm migrate 100 pve2 --online

# Suspend (minimal downtime)
qm migrate 100 pve2 --suspend
```

## Advanced Migration

### SSL/TLS Migration

```bash
# Use encrypted migration
qm migrate 100 pve2 --migration_network 192.168.1.0/24

# Or use stunnel
pve-migrate -t stunnel
```

### Migration via NAT

```bash
# Configure NAT in /etc/pve/datacenter.conf
migration: stunnel

# Enable NAT
pvesh put /cluster/options -migration_network 192.168.1.0/24
```

## Post-Migration

### Verify VM

```bash
# Check VM status
qm status 100

# Check resources
qm monitor 100
```

### Cleanup

```bash
# Verify migration complete
qm list

# Remove old disk if needed
# (only for offline migration)
```

## Troubleshooting

### Migration Fails

```bash
# Check network
ping pve2

# Check storage
pvesm status

# Check logs
tail -f /var/log/pveproxy/access.log
```

### Common Issues

| Error | Solution |
|-------|----------|
| Timeout | Increase timeout |
| Storage not shared | Use shared storage |
| Network blocked | Open port 8006 |
| Memory | Reduce VM memory |

## Keywords

#migration #live-migration #online-migration #cluster #ha #failover

## Related Articles

- [[Advanced]]
- [[Cluster]]
- [[HA-Deep-Dive]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
