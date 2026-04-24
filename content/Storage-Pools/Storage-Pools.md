---
title: Storage Pools
---

# Storage Pools

## Create Pool

```bash
# Create directory pool
pvesm add dir local --path /var/lib/vz

# Create NFS pool
pvesm add nfs nfs1 --server 192.168.1.200 --export /share
```

## Commands

| Command | Description |
|---------|-------------|
| `pvesm add` | Add storage |
| `pvesm remove` | Remove storage |
| `pvesm status` | Show status |

## See Also

- [[index|Back to Proxmox VE]]