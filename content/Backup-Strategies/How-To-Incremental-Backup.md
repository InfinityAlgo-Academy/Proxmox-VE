# How to Implement Incremental Backups - Complete Guide

## Question: How do I save backup space with incremental backups?

### Answer: Use Proxmox Backup Server or differential backup

---

## Understanding Backup Types

### Full Backup
- Copies entire VM every time
- Largest storage needed
- Simplest to restore

### Incremental Backup
- First backup is full
- Subsequent backups only changed data
- Requires PBS for true incremental
- Small storage footprint

### Differential Backup
- Tracks changes since last full backup
- Smaller than full, larger than incremental
- Easy to restore

---

## How to Use Incremental with PBS

### Step 1: Install PBS
```bash
apt install proxmox-backup-server
```

### Step 2: Configure in Proxmox
```bash
pvesm add proxmox-backup-server pbs01 \
  --server 192.168.1.250 \
  --datastore main
```

### Step 3: Create Backup Job
1. **Datacenter** → **Backup** → **Add**
2. Select **PBS storage**
3. Enable **Incremental** option

---

## How to Schedule Differential Backup (Without PBS)

### Using Script
```bash
#!/bin/bash
# differential-backup.sh

DATE=$(date +%Y%m%d)
LOCAL="/var/lib/vz/dump"
REMOTE="/mnt/backup"

# Weekly full backup
if [ $(date +%u) -eq 1 ]; then
    echo "Full backup"
    vzdump --mode suspend --vmid all --storage local --compress zstd
    rsync -avz $LOCAL $REMOTE/full-$DATE
else
    echo "Differential backup"
    # Copy only changed files
    rsync -avz --compare-dest=$REMOTE/full-$(date -d "last monday" +%Y%m%d) $LOCAL $REMOTE/diff-$DATE
fi
```

---

## Keywords

#incremental #backup #how-to #pbs #differential

## Related Articles

- [[Backup]]
- [[How-To-Use-PBS]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
