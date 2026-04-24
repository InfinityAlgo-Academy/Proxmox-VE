---
title: Scheduled Tasks
---

# Scheduled Tasks

## Cron Jobs

```bash
# Add cron job
crontab -e

# Example
0 2 * * * /root/backup.sh
```

## Proxmox Scheduled Tasks

```bash
# Schedule VM start
qm start 100 --schedule daily
```

## See Also

- [[index|Back to Proxmox VE]]