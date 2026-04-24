---
title: EFI Configuration
---

# EFI Configuration

## Enable EFI

```bash
# Enable EFI
qm set 100 --bios ovmf

# Set EFI disk
qm set 100 --efidisk0 local:1M
```

## Secure Boot

```bash
# Enable secure boot
qm set 100 --secureboot 1
```

## See Also

- [[index|Back to Proxmox VE]]