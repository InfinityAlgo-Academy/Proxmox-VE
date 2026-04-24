---
title: Services
---

# Services

## Proxmox Services

```bash
# Restart service
systemctl restart pveproxy
systemctl restart pvedaemon

# Check status
systemctl status pveproxy
systemctl status corosync
```

## Autostart

```bash
# Enable autostart
qm start 100 --onboot 1
pct start 200 --onboot 1
```

## See Also

- [[Startup-Services]]
- [[Auto-Start]]
- [[Services]]
[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
