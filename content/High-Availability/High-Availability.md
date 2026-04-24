---
title: High Availability
---

# High Availability

## Overview

HA provides automatic failover for VMs and containers when a node fails.

## Enable HA

```bash
# Enable HA for VM
ha-manager enable vm:100 --group production

# Disable HA
ha-manager disable vm:100

# Show HA status
ha-manager status
```

## HA Commands

| Command | Description |
|---------|-------------|
| `ha-manager status` | Show HA status |
| `ha-manager enable vm:100` | Enable HA |
| `ha-manager disable vm:100` | Disable HA |

## See Also

- [[index|Back to Proxmox VE]]