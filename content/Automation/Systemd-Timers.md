# Systemd Timers for Automation

## Overview

Systemd timers are modern cron alternative offering more features. Timers can be used for scheduling tasks in Proxmox VE.

## Timer Basics

### Creating Timer Unit

```bash
# Create timer file
nano /etc/systemd/system/backup.timer

[Unit]
Description=Backup Timer

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

### Creating Service

```bash
nano /etc/systemd/system/backup.service

[Unit]
Description=Backup Service

[Service]
Type=oneshot
ExecStart=/root/backup.sh

[Install]
WantedBy=multi-user.target
```

## Timer Options

### Schedule Types

```ini
# Daily at 2 AM
OnCalendar=*-*-* 02:00:00

# Weekly Sunday at 2 AM
OnCalendar=Sun *-*-* 02:00:00

# Monthly 1st
OnCalendar=*-*-01 02:00:00

# Every 15 minutes
OnCalendar=*:0/15

# Specific day
OnCalendar=2025-12-25 02:00:00
```

### Timer Features

```ini
# Persistent - run missed if system was down
Persistent=true

# Randomized delay
RandomizedDelaySec=1h

# Wake system
WakeBoot=false
```

## Managing Timers

### Enable Timer

```bash
# Reload systemd
systemctl daemon-reload

# Enable timer
systemctl enable backup.timer

# Start timer
systemctl start backup.timer

# Check status
systemctl status backup.timer
```

### List Timers

```bash
# List all timers
systemctl list-timers

# List timers with details
systemctl list-timers --all
```

## Examples

### Backup Timer

```ini
# /etc/systemd/system/vm-backup.timer
[Unit]
Description=VM Backup Timer

[Timer]
OnCalendar=Mon,Thu *-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/vm-backup.service
[Unit]
Description=VM Backup Service
After=network-online.target

[Service]
Type=oneshot
ExecStart=/root/scripts/vm-backup.sh

[Install]
WantedBy=multi-user.target
```

### Cleanup Timer

```ini
# /etc/systemd/system/cleanup.timer
[Unit]
Description=Cleanup Timer

[Timer]
OnCalendar=Sun *-*-* 03:00:00

[Install]
WantedBy=timers.target
```

## Troubleshooting

### Timer Not Running

```bash
# Check status
systemctl status backup.timer

# Check logs
journalctl -u backup.timer

# Check service
systemctl status backup.service
```

### Run Immediately

```bash
# Start service manually
systemctl start backup.service

# Or trigger timer
systemctl start backup.timer
```

## Keywords

#systemd #timer #automation #scheduling #service

## Related Articles

- [[Automation]]
- [[Cron-Jobs]]
- [[Scripts]]