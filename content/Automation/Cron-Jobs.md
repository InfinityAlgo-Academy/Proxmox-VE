# Cron Jobs Complete Guide

## Overview

Cron is a time-based job scheduler for automatating scripts and commands. Essential for backups, maintenance, and monitoring tasks.

## Basic Cron Syntax

### Understanding Cron Expression

```
┌───────────── minute (0 - 59)
│ ┌─────────── hour (0 - 23)
│ │ ┌──────── day of month (1 - 31)
│ │ │ ┌────── month (1 - 12)
│ │ │ │ ┌──── day of week (0 - 6) (Sunday=0)
│ │ │ │ │
* * * * * command
```

## Cron Examples

### Basic Examples

```cron
# Every minute
* * * * * /root/script.sh

# Every hour
0 * * * * /root/script.sh

# Every day at 2 AM
0 2 * * * /root/script.sh

# Every Monday at 3 AM
0 3 * * 1 /root/script.sh

# Every 15 minutes
*/15 * * * * /root/script.sh
```

### Common Schedules

```cron
# Daily at 2 AM
0 2 * * * /root/backup.sh

# Weekly Sunday at 2 AM
0 2 * * 0 /root/cleanup.sh

# Monthly 1st at 2 AM
0 2 1 * * /root/monthly.sh

# Every 5 minutes
*/5 * * * * /root/monitor.sh

# Every 30 minutes
*/30 * * * * /root/check.sh
```

## Managing Cron Jobs

### Add Cron Job

```bash
# Edit crontab
crontab -e

# Or use specific file
crontab /root/my-cronjobs

# System cron (all users)
nano /etc/crontab
```

### List Cron Jobs

```bash
# Current user crontab
crontab -l

# Specific user
crontab -u root -l

# System cron
cat /etc/crontab
```

### Remove Cron Job

```bash
# Remove all
crontab -r

# Remove without confirmation
crontab -ir
```

## Cron Environment

### PATH Setting

```bash
# Set PATH in crontab
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Or absolute path
0 2 * * * /usr/local/bin/backup.sh
```

### Output Handling

```bash
# Redirect output
0 2 * * * /root/script.sh > /root/script.log 2>&1

# Send email (if configured)
0 2 * * * /root/script.sh

# Discard output
0 2 * * * /root/script.sh > /dev/null 2>&1
```

## Special Characters

### Characters Meaning

| Character | Meaning |
|-----------|---------|
| * | Any value |
| , | Value list (1,2,3) |
| - | Range (1-5) |
| / | Step (*/5) |

### Examples

```cron
# 1, 2, or 3
0 2 * * 1,3,5 /root/script.sh

# 1 through 5
0 2 * * 1-5 /root/script.sh

# Every 2 hours
0 */2 * * * /root/script.sh

# Every 30 minutes
0,30 * * * * /root/script.sh
```

## Proxmox Specific Cron

### Backup Jobs

```cron
# Daily backup at 2 AM
0 2 * * * vzdump --mode suspend --vmid 100 --storage local

# Multiple VMs
0 2 * * * for vm in 100 101 102; do vzdump --mode suspend --vmid $vm --storage local; done
```

### Maintenance Jobs

```cron
# Clean old logs daily
0 3 * * * find /var/log -name "*.log" -mtime +30 -delete

# Update package index
0 4 * * * apt update -y

# Check VM status
*/15 * * * * qm list | grep running
```

## Troubleshooting Cron

### Not Running

```bash
# Check cron service
systemctl status cron

# Check logs
grep CRON /var/log/syslog

# Check mail
mail
```

### Permission Issues

```bash
# Make script executable
chmod +x /root/script.sh

# Check ownership
chown root:root /root/script.sh
```

## Best Practices

### Script Template

```bash
#!/bin/bash
# backup.sh - Backup script

# Exit on error
set -e

# Log
exec > >(logger -t backup) 2>&1

echo "Starting backup..."

# Your commands here
vzdump --mode suspend --vmid 100 --storage local

echo "Backup complete"
```

### Lock Files

```bash
#!/bin/bash
# Prevent multiple instances

LOCKFILE=/var/lock/backup.lock

if [ -f $LOCKFILE ]; then
    echo "Already running"
    exit 1
fi

touch $LOCKFILE
trap "rm -f $LOCKFILE" EXIT

# Your commands here
```

## Keywords

#cron #scheduler #automation #crontab #cronjob

## Related Articles

- [[Automation]]
- [[Scripts]]
- [[Scheduled-Tasks]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
