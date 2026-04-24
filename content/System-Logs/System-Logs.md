---
title: System Logs
---

# System Logs

## View Logs

```bash
# Proxmox logs
journalctl -u pveproxy -n 50

# Kernel logs
dmesg

# All logs
ls -la /var/log/
```

## Log Files

| File | Description |
|-----|-------------|
| `/var/log/pveproxy/` | Web logs |
| `/var/log/pvedaemon/` | API logs |
| `/var/log/syslog` | System logs |

## See Also

- [[index|Back to Proxmox VE]]