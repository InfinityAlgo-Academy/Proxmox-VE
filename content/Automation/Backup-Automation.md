# Backup Automation Scripts

## Overview

Automated backup scripts for Proxmox VE. Schedule, manage, and verify backups automatically.

## Basic Backup Script

### Simple Backup

```bash
#!/bin/bash
# backup-vm.sh - Backup single VM

VMID=$1
STORAGE=${2:-local}
MODE=${3:-suspend}

# Check VM exists
qm list | grep -q "^$VMID " || { echo "VM $VMID not found"; exit 1; }

# Start backup
vzdump --mode $MODE --vmid $VMID --storage $STORAGE

echo "Backup complete for VM $VMID"
```

### Multi-VM Backup

```bash
#!/bin/bash
# backup-all.sh - Backup all VMs

STORAGE=local
LOG="/var/log/backup.log"

# Get running VMs
VM_LIST=$(qm list | grep running | awk '{print $1}')

for VMID in $VM_LIST; do
    echo "[$(date)] Starting backup VM $VMID"
    vzdump --mode suspend --vmid $VMID --storage $STORAGE 2>&1 | tee -a $LOG
    sleep 30
done

echo "[$(date)] All backups complete" | tee -a $LOG
```

## Backup with Verification

### Verify Backup

```bash
#!/bin/bash
# backup-verify.sh - Backup and verify

VMID=$1
STORAGE=local

# Create backup
echo "Creating backup..."
vzdump --mode suspend --vmid $VMID --storage $STORAGE

# Get latest backup
LATEST=$(pvesm list $STORAGE | grep vzdump.*$VMID | tail -1 | awk '{print $2}')

# Verify
if [ -n "$LATEST" ]; then
    echo "Backup verified: $LATEST"
else
    echo "ERROR: Backup verification failed"
    exit 1
fi
```

## Retention Management

### Clean Old Backups

```bash
#!/bin/bash
# cleanup-backups.sh - Remove old backups

STORAGE=local
RETENTION_DAYS=7

# Find old backups
OLD=$(pvesm list $STORAGE | grep vzdump | \
  awk -v days=$RETENTION_DAYS \
  'age($6) > days {print $2}')

for backup in $OLD; do
    echo "Removing: $backup"
    pvesm remove $STORAGE $backup
done

echo "Cleanup complete"
```

## Encrypted Backups

### Encrypted Backup

```bash
#!/bin/bash
# backup-encrypted.sh - Encrypted backup

VMID=$1
STORAGE=local
PASSFILE="/root/backup.pass"

# Create encrypted backup
vzdump --mode suspend \
  --vmid $VMID \
  --storage $STORAGE \
  --compress zstd \
  --encryption $PASSFILE

echo "Encrypted backup complete"
```

## Restore Automation

### Automated Restore

```bash
#!/bin/bash
# restore-vm.sh - Restore from backup

BACKUP_ID=$1
TARGET_VMID=$2
STORAGE=local

# Stop target if running
qm stop $TARGET_VMID --forceStop 1 2>/dev/null || true

# Restore
vzdump --restore $BACKUP_ID --vmid $TARGET_VMID

echo "Restore complete"
```

## Full Backup Solution

### Complete Script

```bash
#!/bin/bash
# proxmox-backup.sh - Complete backup solution

set -euo pipefail

STORAGE="local"
RETENTION=7
LOG_FILE="/var/log/proxmox-backup.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a $LOG_FILE
}

# Get VMs to backup (running or all)
VM_LIST="${*:-$(qm list | grep running | awk '{print $1}')}"

for VMID in $VM_LIST; do
    log "Backing up VM $VMID..."
    
    # Create backup
    vzdump --mode suspend \
      --vmid $VMID \
      --storage $STORAGE \
      --compress zstd \
      --mailto admin@example.com
    
    log "VM $VMID backup complete"
done

# Cleanup old
log "Cleaning old backups..."
find /var/lib/vz/dump -name "vzdump*.tar.zst" -mtime +$RETENTION -delete

log "All backups complete"
```

## Keywords

#backup #automation #restore #retention #scheduled

## Related Articles

- [[Automation]]
- [[Backup]]
- [[Cron-Jobs]]
---

[[index|Back to Proxmox VE]]
