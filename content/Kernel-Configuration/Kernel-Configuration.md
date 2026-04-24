---
title: Kernel Configuration
---

# Kernel Configuration

## Update Kernel

```bash
# Update kernel
apt update && apt install pve-kernel

# Set default kernel
proxkmo-set-default-kernel 5.13.8-1-pve
```

## Kernel Parameters

```bash
# Edit GRUB
nano /etc/default/grub

# Update GRUB
update-grub
```

## See Also

- [[index|Back to Proxmox VE]]