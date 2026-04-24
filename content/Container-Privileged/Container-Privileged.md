---
title: Container Privileged
---

# Privileged Containers

## Create Privileged Container

```bash
# Create privileged container
pct create 200 \
  --ostype debian \
  --rootfs local:10 \
  --unprivileged 0
```

## Security

Privileged containers have full root access to the host. Use with caution.

## See Also

- [[index|Back to Proxmox VE]]