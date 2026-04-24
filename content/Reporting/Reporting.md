---
title: Reporting
---

# Reporting

## Proxmox Reporting

```bash
# Generate report
pvesh get /nodes/localhost/status --output-format json

# Export
pvesh get /cluster/resources --output-format yaml
```

## Reports

- VM usage
- Storage usage
- Cluster status

## See Also

- [[index|Back to Proxmox VE]]