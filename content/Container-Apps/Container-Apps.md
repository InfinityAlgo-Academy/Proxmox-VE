---
title: Container Apps
---

# Container Apps

## Popular LXC Apps

### Docker

```bash
# Install Docker in container
pct exec 200 -- apt-get update
pct exec 200 -- apt-get install -y docker.io
pct exec 200 -- systemctl start docker
```

### Home Assistant

```bash
# Install Home Assistant
pct exec 200 -- apt-get install -y software-properties-common
pct exec 200 -- add-apt-repository universe
pct exec 200 -- apt-get update
pct exec 200 -- apt-get install -y hassio-db-setup
```

### Pi-hole

```bash
# Install Pi-hole
pct exec 200 -- curl -L https://install.pi-hole.net | bash
```

## Common Apps

| App | Port | Description |
|-----|------|-------------|
| Plex | 32400 | Media server |
| Jellyfin | 8096 | Media server |
| AdGuard | 3000 | Ad blocker |
| Nginx | 80 | Reverse proxy |

## See Also

- [[Containers]]
- [[Docker]]
- [[Home-Automation]]
[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
