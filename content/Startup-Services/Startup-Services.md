---
title: Startup Services
---

# Startup Services

## autostart

```bash
# Enable VM autostart
qm set 100 --onboot 1

# Enable container autostart
pct start 200 --onboot 1
```

## Service Order

```bash
# Set start order
qm set 100 --startup 1
pct set 200 --startup 2
```

## See Also

- [[index|Back to Proxmox VE]]