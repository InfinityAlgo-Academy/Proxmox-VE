---
title: Virtual Disk
---

# Virtual Disk

## Create Disk

```bash
# Add disk to VM
qm set 100 --scsi1 local:32G

# Resize disk
qm resize 100 scsi0 +10G
```

## Disk Types

| Type | Description |
|-----|-------------|
| `scsi0` | SCSI |
| `sata0` | SATA |
| `virtio0` | VirtIO |

## See Also

- [[index|Back to Proxmox VE]]