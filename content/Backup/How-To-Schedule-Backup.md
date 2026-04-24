# How to Schedule Automatic Backups - Complete Guide

## Question: How do I schedule automatic backups in Proxmox VE?

### Answer: Multiple options available

---

## Option 1: Web UI Scheduled Jobs (Easiest)

### Step-by-Step:

**1. Go to Backup Section**
- Navigate to **Datacenter** → **Backup**

**2. Create New Job**
- Click **Add**

**3. Configure Schedule**
Choose your frequency:
- **Daily** - Runs every day
- **Weekly** - Runs once a week (Sunday)
- **Monthly** - Runs 1st of month
- **Custom** - Use cron expression

**4. Set Time**
- Default: 2:00 AM (off-peak)
- Click time to change

**5. Select VMs/Containers**
- Choose "Include specific VMs"
- Select all or specific VMs

**6. Save Job**
- Click **Create**
- Job appears in list - enabled by default

---

## Option 2: Custom Cron Schedule

### Common Schedules

```bash
# Edit crontab
crontab -e

# Add these lines:

# Daily at 2 AM
0 2 * * * vzdump --mode snapshot --vmid all --storage local --compress lzo

# Every 6 hours
0 */6 * * * vzdump --mode snapshot --vmid all --storage local

# Monday, Wednesday, Friday at 2 AM
0 2 * * 1,3,5 vzdump --mode snapshot --vmid all --storage local

# Weekly Sunday at 3 AM (full backup)
0 3 * * 0 vzdump --mode suspend --vmid all --storage local --compress zstd
```

### Cron Expression Guide

| Field | Values | Example |
|-------|--------|----------|
| Minute | 0-59 | 0 = top of hour |
| Hour | 0-23 | 2 = 2 AM |
| Day | 1-31 | * = every day |
| Month | 1-12 | * = every month |
| Day of week | 0-6 | 0 = Sunday |

---

## Option 3: Systemd Timers (Modern)

### Create Timer

```bash
# Create service
nano /etc/systemd/system/pvebackup.service

[Unit]
Description=Proxmox Backup

[Service]
Type=oneshot
ExecStart=/usr/bin/vzdump --mode snapshot --vmid all --storage local
```

### Create Timer

```bash
nano /etc/systemd/system/pvebackup.timer

[Unit]
Description=Backup Timer

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

### Enable Timer

```bash
systemctl daemon-reload
systemctl enable pvebackup.timer
systemctl start pvebackup.timer
```

---

## How to Set Retention (Keep Backups)

### Using Web UI

1. Backup Job → **Edit**
2. Find **Retention**
3. Set number:
   - **Daily**: 7 (keep 7 days)
   - **Weekly**: 4 (keep 4 weeks)
   - **Monthly**: 3 (keep 3 months)

### Using CLI

```bash
# Auto-cleanup with job
vzdump --mode snapshot --vmid all --storage local \
  --prune-backup keep-daily=7,keep-weekly=4,keep-monthly=3
```

---

## How to Verify Schedule is Working

```bash
# Check scheduled task
systemctl list-timers | grep backup

# Check logs
journalctl -u vzdump -n 20
```

---

## Troubleshooting Scheduling Issues

### Backup Not Running
```bash
# Check cron service
systemctl status cron

# Enable cron
systemctl enable cron
```

### Wrong Time
```bash
# Check timezone
timedatectl

# Set correct timezone
timedatectl set-timezone America/New_York
```

---

## Pro Tips

1. **Off-peak hours**: 1-5 AM best
2. **Test first**: Manual backup before scheduling
3. **Monitor**: Enable email notifications

---

## Keywords

#schedule #cron #how-to #automation #backup

## Related Articles

- [[Backup]]
- [[How-To-Configure-Backup]]
- [[Cron-Jobs]]