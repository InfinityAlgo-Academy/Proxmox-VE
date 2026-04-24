# How to Create a Backup Strategy - Complete Beginner's Guide

## Question: How do I create an effective backup strategy?

### Answer: Follow the 3-2-1 Rule

---

## Understanding the 3-2-1 Backup Rule

### What is 3-2-1?
- **3** copies of your data
- **2** different storage types
- **1** copy offsite

### Why It Works
This rule ensures you have redundant copies. If one storage fails, you have backups in other locations.

---

## How to Implement 3-2-1

### Step 1: Identify Primary Data
```bash
# List all VMs and containers to backup
qm list
pct list
```

### Step 2: Create 3 Copies Locally
```bash
# Copy 1: Primary storage
vzdump --mode snapshot --vmid 100 --storage local

# Copy 2: Different storage type
vzdump --mode snapshot --vmid 100 --storage nfs-backup

# Copy 3: External USB
vzdump --mode snapshot --vmid 100 --storage usb-backup
```

### Step 3: Set Up Offsite Backup
```bash
# Option A: rsync to remote server
rsync -avz /var/lib/vz/dump/ user@remote:/backup/

# Option B: Cloud backup
rclone sync /var/lib/vz/dump/ cloud:backup-bucket
```

---

## Complete Strategy Example

### Daily Backups (Copies 1 & 2)
```bash
# Job 1: Local storage
# Job 2: NFS storage
# Schedule: Daily at 2 AM
```

### Weekly Backup (Copy 3)
```bash
# Job 3: USB drive
# Schedule: Sunday at 3 AM
# Remove USB after backup for security
```

### Monthly Offsite
```bash
# rsync to remote on 1st of month
0 3 1 * * rsync -avz /var/lib/vz/dump/ user@remote:/backup/
```

---

## Keywords

#backup-strategy #how-to #321-rule #beginner #planning

## Related Articles

- [[Backup]]
- [[How-To-Configure-Backup]]
- [[Proxmox-VE]]