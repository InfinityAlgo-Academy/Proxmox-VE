# How to Set Backup Retention - Complete Guide

## Question: How long should I keep backups?

### Answer: Use retention policy - never keep forever

---

## Understanding Backup Retention

### Retention Options

| Type | Typical Keep | Use Case |
|------|-------------|----------|
| Daily | 7 days | Recent recovery |
| Weekly | 4 weeks | Point-in-time recovery |
| Monthly | 3-6 months | Archive compliance |
| Yearly | 1-2 years | Long-term archive |

---

## How to Set Retention (Web UI)

### Step-by-Step:

**1. Edit Backup Job**
- Go to **Datacenter** → **Backup**
- Click on job → **Edit**

**2. Configure Retention**
Find these options:
- **Keep Daily**: 7 (keep 7 daily backups)
- **Keep Weekly**: 4 (keep 4 weekly backups)
- **Keep Monthly**: 3 (keep 3 monthly backups)

**3. Save**
- Click **OK**

---

## How to Set Retention (CLI)

```bash
# Set retention for backup job
vzdump --mode snapshot --vmid all --storage local \
  --prune-backup keep-daily=7,keep-weekly=4,keep-monthly=3
```

### Prune Options Explained

| Option | Description |
|--------|-------------|
| keep-daily=7 | Keep 7 daily backups |
| keep-weekly=4 | Keep 4 weekly backups |
| keep-monthly=3 | Keep 3 monthly backups |

---

## How to Manually Clean Old Backups

### Using find Command

```bash
# Delete backups older than 7 days
find /var/lib/vz/dump -name "vzdump*.tar.*" -mtime +7 -delete

# Delete backups older than 30 days
find /var/lib/vz/dump -name "vzdump*.tar.*" -mtime +30 -delete
```

---

## Recommended Retention Policies

### Home Lab
```bash
# Keep 3 daily, 2 weekly
keep-daily=3,keep-weekly=2
```

### Small Business
```bash
# Keep 7 daily, 4 weekly, 3 monthly
keep-daily=7,keep-weekly=4,keep-monthly=3
```

### Enterprise
```bash
# Keep 7 daily, 5 weekly, 12 monthly
keep-daily=7,keep-weekly=5,keep-monthly=12
```

---

## Keywords

#retention #how-to #clean #policy #storage

## Related Articles

- [[Backup]]
- [[How-To-Configure-Backup]]
- [[How-To-Backup-Storage]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
