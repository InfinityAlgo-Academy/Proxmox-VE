# Proxmox VE Cloud Integration Guide

## Table of Contents

1. [Cloud Backup](#cloud-backup)
2. [Cloud Sync](#cloud-sync)
3. [Cloud Migration](#cloud-migration)
4. [Hybrid Cloud](#hybrid-cloud)

---

## Cloud Backup

### Backblaze

```bash
# Install rclone
curl https://rclone.org/install.sh | sudo bash

# Configure
rclone config

# Sync backups
rclone sync /mnt/backup/dump/ backblaze:ProxmoxBackups
```

### AWS S3

```bash
# Configure S3
rclone config

# Sync
rclone sync /mnt/backup/dump/ s3:proxmox-backups
```

### Google Drive

```bash
# Configure
rclone config

# Sync
rclone sync /mnt/backup/dump/ gdrive:ProxmoxBackups
```

---

## Cloud Sync

### rsync to Cloud

```bash
# Google Drive
rclone sync /mnt/pve/ gdrive:Proxmox

# Backblaze
rclone sync /mnt/pve/ backblaze:Proxmox

# AWS S3
rclone sync /mnt/pve/ s3:proxmox
```

---

## Hybrid Cloud

### Connect to Cloud

```bash
# VPN to cloud
wg-quick up wg0

# Access cloud resources
ssh cloud-server
```

---

## Keywords

#cloud #backup #sync #hybrid

## Links

- [[Proxmox-VE]] - Main documentation
- [[Backup]] - Backup configuration
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
