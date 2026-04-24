---
title: LXC Templates
---

# LXC Templates

## Download Templates

```bash
# Download template
pvesm add dir vztmpl --path /var/lib/vz/template/cache
wget -P /var/lib/vz/template/cache https://download.proxmox.com/images/

## List Templates

```bash
# List available
pvesm list local
pct template list
```

## See Also

- [[index|Back to Proxmox VE]]