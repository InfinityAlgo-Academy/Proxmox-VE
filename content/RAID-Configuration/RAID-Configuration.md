---
title: RAID Configuration
---

# RAID Configuration

## RAID Levels

| Level | Disks | Description |
|-------|------|--------------|
| RAID 0 | 2 | Striping (no redundancy) |
| RAID 1 | 2 | Mirroring |
| RAID 5 | 3 | Parity |
| RAID 6 | 4 | Double parity |
| RAID 10 | 4 | Mirror + Stripe |

## Create RAID

```bash
# Install mdadm
apt install mdadm -y

# Create RAID
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda /dev/sdb

# Check status
cat /proc/mdstat
mdadm --detail /dev/md0
```

## See Also

- [[Storage]]
- [[Disk-Management]]
- [[ZFS-Tuning]]
[[index|Back to Proxmox VE]]
