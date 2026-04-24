---
title: VM Migration Live
---

# Live Migration

## Migrate VM

```bash
# Live migrate VM
qm migrate 100 node2 --online

# Offline migrate
qm migrate 100 node2 --offline
```

## Requirements

- Shared storage
- Same CPU type (or compatible)
- Network connectivity

## See Also

- [[index|Back to Proxmox VE]]