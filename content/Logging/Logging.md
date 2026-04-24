---
title: Logging
---

# Logging

## System Logs

```bash
# View logs
journalctl -xe
journalctl -u pveproxy
journalctl -u pvedaemon
```

## Log Files

| Path | Description |
|------|-------------|
| `/var/log/pveproxy/` | Web UI logs |
| `/var/log/pvedaemon/` | API logs |
| `/var/log/syslog` | System logs |

## See Also

- [[index|Back to Proxmox VE]]