---
title: Resource Management
---

# Resource Management

## CPU

```bash
# Limit CPU
qm set 100 --cores 2
qm set 100 --cpulimit 2
```

## Memory

```bash
# Limit memory
qm set 100 --memory 4096
qm set 100 --memory 4096 --balloon 2048
```

## See Also

- [[index|Back to Proxmox VE]]