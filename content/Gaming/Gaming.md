---
title: Gaming
---

# Gaming on Proxmox

## Gaming VMs

### Windows Gaming VM

```bash
# Create Gaming VM
qm create 100 --name "gaming" --memory 16384 --cores 8
qm set 100 --cpu host
qm set 100 --vga virtio
qm set 100 --hostpci0 01:00,pcie=1
```

## GPU Passthrough

- [[PCI-Passthrough]]

## See Also

- [[index|Back to Proxmox VE]]