---
title: Cloud Apps
---

# Cloud Apps

## Popular Cloud Apps on Proxmox

### web-based apps

- Nextcloud - File sharing
- Jellyfin - Media server
- AdGuard Home - Ad blocker

### Development

- GitLab - Version control
- Jenkins - CI/CD
- Portainer - Docker management

## Installation

```bash
# Install via CT
pct create 200 local:vztmpl/debian-12.tar.gz
pct start 200
pct exec 200 -- apt install -y curl wget
```

## See Also

- [[index|Back to Proxmox VE]]