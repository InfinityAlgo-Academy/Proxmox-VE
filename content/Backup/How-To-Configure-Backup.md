# How to Configure Proxmox VE Backup - Complete Beginner's Guide

## Question: How do I set up automatic backups in Proxmox VE?

### Answer: Follow these simple steps

#### Method 1: Using Web UI (Recommended for Beginners)

**Step 1: Access Backup Configuration**
1. Log in to Proxmox web interface
2. Navigate to **Datacenter** → **Backup**

**Step 2: Create New Backup Job**
1. Click **Add** button
2. Fill in the following:
   - **ID**: backup-daily
   - **Target Storage**: Select local or NFS
   - **Schedule**: Choose schedule (e.g., "Daily")
   - **Selection Mode**: Select VMs/Containers

**Step 3: Choose What to Backup**
1. Select specific VMs or use "All"
2. Example: Check VMs 100, 101, 102

**Step 4: Configure Backup Options**
1. **Mode**: snapshot (no downtime)
2. **Compression**: lzo (fast) or zstd (better)
3. **Email**: Enable notification

**Step 5: Save and Enable**
1. Click **Create**
2. Enable job with toggle switch

### Method 2: Using CLI

```bash
# Create daily backup job
vzdump 100 --mode snapshot --storage local --compress lzo

# For multiple VMs
vzdump 100 101 102 --mode snapshot --storage local
```

### Method 3: Using Cron (Advanced)

```bash
# Edit crontab
crontab -e

# Add backup job
# Daily at 2 AM
0 2 * * * /usr/bin/vzdump --mode snapshot --vmid all --storage local

# Weekly Sunday at 2 AM
0 2 * * 0 /usr/bin/vzdump --mode suspend --vmid all --storage local
```

---

## Understanding Backup Modes

| Mode | Downtime | Best For |
|------|----------|----------|
| **snapshot** | None | Running VMs |
| **suspend** | Brief(~30s) | Critical VMs |
| **stopped** | Full | Downtime OK |

---

## How to Check if Backup is Working

```bash
# Check backup log
vzdump --vmid 100 --log

# Verify backup exists
ls -la /var/lib/vz/dump/

# Check storage
pvesm status local
```

---

## Troubleshooting Common Issues

### Backup Failed
```bash
# Check available space
pvesm status

# Check VM exists
qm list
```

### Storage Full
```bash
# Clean old backups
rm -rf /var/lib/vz/dump/*

# Or configure retention
# Web UI → Backup → Job → Retention
```

---

## Pro Tips

1. **Schedule backups for off-peak hours** (2 AM typical)
2. **Use compression** to save space
3. **Test restore** monthly
4. **Enable email notifications**

---

## Keywords

#backup #how-to #beginner #vzdump #configuration

## Related Articles

- [[Backup]]
- [[Backup-Strategies]]
- [[Disaster-Recovery]]
---

[[index|Back to Proxmox VE]]
