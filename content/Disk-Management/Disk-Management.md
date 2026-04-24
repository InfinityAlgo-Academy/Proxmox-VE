---
title: Disk Management
---

# Disk Management

## List Disks

```bash
lsblk
fdisk -l
smartctl -a /dev/sda
```

## Partition Disk

```bash
fdisk /dev/sdb
# n (new partition)
# w (write)
```

## See Also

- [[Storage]]
- [[RAID-Configuration]]
- [[LVM-Management]]
[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
