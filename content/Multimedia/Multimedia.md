---
title: Multimedia
---

# Multimedia

## Media Servers

### Jellyfin

```bash
# Install Jellyfin
pct create 200 --ostype debian
pct exec 200 -- curl -L https://repo.jellyfin.org/install.sh | bash
```

### Emby

```bash
# Install Emby
pct exec 200 -- wget https://app.emby.media/stabl... && dpkg -i emby-server*.deb
```

## See Also

- [[index|Back to Proxmox VE]]