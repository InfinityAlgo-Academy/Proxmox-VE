---
title: LVM Thin
---

# LVM Thin

## Overview

LVM Thin provides thin provisioning for storage pools.

## Create Thin Pool

```bash
# Create physical volume
pvcreate /dev/sdb

# Create volume group
vgcreate vg_data /dev/sdb

# Create thin pool
lvcreate -L 100G --thinpool thin_pool vg_data
```

## Commands

| Command | Description |
|---------|-------------|
| `lvs` | List logical volumes |
| `lvdisplay` | Display LV info |
| `lvremove` | Remove LV |

## See Also

- [[LVM-Management]]
- [[Storage]]
- [[ZFS-Tuning]]
[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
