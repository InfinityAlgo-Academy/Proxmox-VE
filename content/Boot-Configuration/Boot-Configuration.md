---
title: Boot Configuration
---

# Boot Configuration

## UEFI Boot

```bash
# Check boot mode
ls /sys/firmware/efi

# Set UEFI boot
qm set 100 --bios ovmf
```

## Boot Order

```bash
# Set boot order
qm set 100 --boot order=scsi0

# Set boot device
qm set 100 --boot order=scsi0;ide2;net0
```

## See Also

- [[BIOS-Configuration]]
- [[EFI-Configuration]]
- [[VM]]
[[index|Back to Proxmox VE]]
