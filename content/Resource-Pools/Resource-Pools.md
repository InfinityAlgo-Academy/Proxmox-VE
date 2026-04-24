---
title: Resource Pools
---

# Resource Pools

## Create Pool

```bash
# Create resource pool
qm set 100 -- cores 2 --memory 4096

# Set CPU limit
qm set 100 --cores 2

# Set memory limit  
qm set 100 --memory 4096
```

## Manage Pools

```bash
# List pools
qm pool list

# Add to pool
qm pool add production 100
```

## See Also

- [[index|Back to Proxmox VE]]