---
title: IoT
---

# IoT on Proxmox

## IoT Platforms

### Home Assistant

```bash
# Install HASS
pct create 200 local:vztmpl/debian-12.tar.gz
pct exec 200 -- curl -sL https://raw.githubusercontent.com/_homeassistant/creator/master/install/linux | bash
```

### Pi-hole

```bash
# Install Pi-hole
pct exec 200 -- curl -L https://install.pi-hole.net | bash
```

## See Also

- [[index|Back to Proxmox VE]]