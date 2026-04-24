---
title: Scheduled Tasks Complete Guide
---

# Scheduled Tasks Complete Guide

## Overview

Proxmox VE built-in scheduled tasks for automated operations like backups, maintenance, and updates.

## Access Scheduled Tasks

### Via Web UI

1. Navigate to **Datacenter → Scheduled Tasks**
2. View, create, or edit tasks

### Via CLI

```bash
# List scheduled tasks
pvesh get /cluster/scheduler

# View task log
pvesh get /cluster/scheduler/log
```

## Create Scheduled Task

### Backup Schedule

```bash
# Create backup schedule
pvesh create /cluster/vzdump \
  --vmid 100 \
  --schedule daily \
  --storage local \
  --mode suspend
```

### Schedule Options

| Value | Description |
|-------|-------------|
| daily | Daily at 2 AM |
| weekly | Sunday 2 AM |
| monthly | 1st of month |
| */n | Every n hours |
| custom | Custom cron |

## Common Tasks

### VM Backup

```bash
# Daily backup at 2 AM
pvesh create /cluster/vzdump \
  --vmid 100 \
  --schedule daily \
  --storage local \
  --mode suspend \
  --compress zstd

# Weekly full backup
pvesh create /cluster/vzdump \
  --vmid 100 \
  --schedule weekly \
  --storage local \
  --mode stop
```

### Update Checks

```bash
# Check for updates daily
# System -> Options -> Update

# Schedule APT
pvesh create /cluster/apt \
  --schedule daily
```

## Manage Tasks

### List Tasks

```bash
# List all scheduled tasks
pvesh get /cluster/scheduler

# Show specific task
pvesh get /cluster/scheduler/100
```

### Delete Task

```bash
# Delete task
pvesh delete /cluster/scheduler/100
```

## Troubleshooting

### Task Not Running

```bash
# Check task log
pvesh get /cluster/scheduler/log

# Check systemd timer
systemctl status pvescheduler
```

## Keywords

#scheduled-tasks #automation #backup #scheduler

## Related Articles

- [[Automation]]
- [[Cron-Jobs]]
- [[Backup]]
---

[[index|Back to Proxmox VE]]
