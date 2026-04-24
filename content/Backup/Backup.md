# Proxmox VE Backup

## Built-in Backup

### Via Web UI
1. **Datacenter** → **Backup** → **Add**
2. **Schedule**: Daily/Weekly
3. **Storage**: Target storage
4. **Selection**: VMs/Containers to backup
5. **Compression**: lzo (fast) or gzip (small)
6. **Mode**: Stopped/Snapshot/Suspend

### Backup Modes

| Mode | Description | Downtime |
|------|-------------|----------|
| stopped | Guest must be stopped | Full |
| suspend | Quiesce disk | Brief |
| snapshot | Live snapshot | None |

### Configuration (CLI)
```bash
# Create backup job
vzdump 100 --storage local --mode snapshot --compress lzo

# Schedule backup
# Add via web UI:
# Datacenter → Backup → Add
```

### Restore
```bash
# List backups
vzdump --list

# Restore from backup
# Web UI: Content → Select backup → Restore
# Or via CLI
qmrestore /mnt/backup/dump/vzdump-qemu-100-2024_01_01-12_00.tar.gz 100
```

## Proxmox Backup Server (PBS)

### Install PBS
```bash
# Install Proxmox Backup Server
apt install proxmox-backup-server -y

# Configure
# Web UI: https://your-pbs:8007
```

### Configure PBS in Proxmox
```bash
# Add PBS as storage
pvesm add proxmox-backup-server pbs01 \
  --server 192.168.1.200 \
  --backup-username backup@pam \
  --fingerprint FINGERPRINT \
  --datastore main
```

### PBS Features
- **Deduplication**: Save storage space
- **Encryption**: Client-side encryption
- **Incremental**: Only changed blocks
- **Integrity**: Provenance verification
- **Pruning**: Automatic cleanup

## Custom Backup Scripts

### Backup Script
```bash
#!/bin/bash
# /root/scripts/backup.sh
BACKUP_DIR="/mnt/backup"
DATE=$(date +%Y%m%d)

# VMs to backup
for VM in 100 101 102; do
  echo "Backing up VM $VM..."
  vzdump $VM --mode snapshot --storage local --compress lzo
done

# Cleanup old backups
find $BACKUP_DIR -mtime +7 -delete

echo "Backup complete!"
```

### Schedule with Cron
```bash
# Edit crontab
crontab -e

# Daily at 2 AM
0 2 * * * /root/scripts/backup.sh
```

## Cloud Backup Options

### rsync to Remote
```bash
# rsync to remote server
rsync -avz /mnt/pve/backup/ user@backup-server:/path/
```

### rclone to Cloud
```bash
# Install rclone
curl https://rclone.org/install.sh | sudo bash

# Configure
rclone config

# Sync to S3/Google Drive/etc
rclone sync /mnt/pve/backup remote:bucket
```

## Restore Procedures

### Full System Restore
```bash
# Reinstall Proxmox
# Import backup
# Restore manually

# Or use PBS one-click restore
# PBS → Content → Select → Restore
```

### VM/Container Restore
```bash
# List available backups
ls /mnt/backup/dump/

# Restore
# Web UI: Datacenter → Backup → Select file → Restore
# Or:
pct restore 100 /mnt/backup/dump/vzdump-lxc-100-2024.tar.gz
qmrestore /mnt/backup/dump/vzdump-qemu-102-2024.tar.gz 102
```

## Backup Best Practices

- **Follow 3-2-1 rule**: 3 copies, 2 media, 1 offsite
- **Test restores**: Regularly verify backups
- **Encrypt**: Use PBS encryption
- **Schedule**: Off-peak hours
- **Monitor**: Check backup logs

## Keywords

#backup #restore #vzdump #pbs #disaster-recovery #cron #rsync

## Links

- [[Proxmox-VE]] - Main documentation
- [[Storage]] - Backup storage
- [[VM]] - VM backup
- [[Containers]] - Container backup
- [[Security]] - Backup encryption
---

[[index|Back to Proxmox VE]]
