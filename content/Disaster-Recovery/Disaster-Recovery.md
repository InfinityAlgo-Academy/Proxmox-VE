---
title: Disaster Recovery
---

# Disaster Recovery

## Backup Strategy

```bash
# Manual backup
vzdump --mode snapshot --vmid 100 --storage local

# Schedule backup
# Datacenter > Backup > Add
```

## Recovery Steps

1. Boot from Proxmox ISO
2. Restore from backup
3. Verify services

## See Also

- [[index|Back to Proxmox VE]]