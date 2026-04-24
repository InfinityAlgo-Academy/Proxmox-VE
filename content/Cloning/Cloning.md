---
title: Cloning
---

# Cloning VMs and Containers

## Clone VM

```bash
# Full clone
qm clone 100 101 --full --name "my-vm-clone"

# Linked clone
qm clone 100 102 --name "my-linked-clone"
```

## Clone Container

```bash
# Clone container
pct clone 200 201 --full --name "my-container-clone"
```

## See Also

- [[index|Back to Proxmox VE]]