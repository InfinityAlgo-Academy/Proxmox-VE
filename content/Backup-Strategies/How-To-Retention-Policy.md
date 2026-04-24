# How to Set Backup Retention Policy - Complete Guide

## Question: How long should I keep backups?

### Answer: Use a retention policy

---

## Understanding Retention

### Retention Options
| Type | Keep | Use Case |
|------|------|----------|
| Daily | 7 days | Recent recovery |
| Weekly | 4 weeks | Last month recovery |
| Monthly | 12 months | Year-end archive |
| Yearly | 7 years | Compliance |

---

## How to Configure Retention (Web UI)

### Step 1: Edit Backup Job
1. Go to **Datacenter** → **Backup**
2. Select your backup job → **Edit**

### Step 2: Set Retention Values
- **Keep Daily**: 7
- **Keep Weekly**: 4
- **Keep Monthly**: 3

### Step 3: Save
Click **OK**

---

## How to Configure Retention (CLI)

### Set in Backup Job
```bash
vzdump --mode snapshot --vmid all --storage local \
  --prune-backup keep-daily=7,keep-weekly=4,keep-monthly=3
```

---

## How to Manually Clean Old Backups

### Using find command
```bash
# Remove backups older than 30 days
find /var/lib/vz/dump -name "vzdump*.tar.*" -mtime +30 -delete

# Remove backups larger than specific size
du -sh /var/lib/vz/dump/*
```

### Cleanup Script
```bash
#!/bin/bash
# cleanup-old-backups.sh

RETENTION_DAYS=30

# Find and remove old files
find /var/lib/vz/dump -name "vzdump*.tar.*" -mtime +$RETENTION_DAYS -delete

echo "Cleaned backups older than $RETENTION_DAYS days"
```

---

## Retention Policy Examples

### Home Lab
```
Daily: 3
Weekly: 2
Monthly: 0
```

### Small Business
```
Daily: 7
Weekly: 4
Monthly: 3
```

### Enterprise (Compliance)
```
Daily: 7
Weekly: 5
Monthly: 12
Yearly: 7
```

---

## Keywords

#retention #backup-policy #how-to #cleanup #storage

## Related Articles

- [[Backup]]
- [[How-To-Retention-Policy]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
