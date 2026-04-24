# Hooks and Event-Driven Automation

## Overview

Hooks allow scripts to run on specific Proxmox VE events. React to VM lifecycle events automatically.

## Hook Scripts

### VM Hook Script

```bash
#!/bin/bash
# vm-hook.sh - Hook script for VM events

VMID=$1
EVENT=$2

log() {
    echo "[$(date)] VM $VMID: $EVENT" >> /var/log/vm-hooks.log
}

case $EVENT in
    pre-start)
        log "VM starting..."
        ;;
    post-start)
        log "VM started"
        # Notify, update status
        ;;
    pre-stop)
        log "VM stopping..."
        ;;
    post-stop)
        log "VM stopped"
        # Cleanup, logging
        ;;
    pre-migrate)
        log "VM migrating..."
        ;;
    post-migrate)
        log "VM migrated"
        ;;
esac
```

### Configure Hook

```bash
# Add hook to VM
qm set 100 --hookscript /root/vm-hook.sh
```

## Use Cases

### Startup Notification

```bash
#!/bin/bash
# notify-startup.sh

VMID=$1

# Send notification (Slack, email, etc)
curl -X POST https://hooks.example.com/notify \
  -d "{\"text\":\"VM $VMID started\"}"

# Update monitoring
echo "$(date): VM $VMID started" >> /var/log/vm-updates.log
```

### Backup Trigger

```bash
#!/bin/bash
# backup-on-stop.sh

VMID=$1
EVENT=$2

if [ "$EVENT" = "pre-stop" ]; then
    echo "Triggering backup for VM $VMID"
    vzdump --mode suspend --vmid $VMID --storage local
fi
```

### Resource Limits

```bash
#!/bin/bash
# adjust-resources.sh

VMID=$1
EVENT=$2

if [ "$EVENT" = "post-start" ]; then
    # Check and adjust CPU limit
    qm set $VMID --cpulimit 50
fi
```

## Event Types

| Event | Description |
|-------|-------------|
| pre-start | Before VM starts |
| post-start | After VM starts |
| pre-stop | Before VM stops |
| post-stop | After VM stops |
| pre-migrate | Before migration |
| post-migrate | After migration |

## Configuration

### Enable Hooks

```bash
# Add hook to VM
qm set 100 --hookscript /path/to/script.sh

# Remove hook
qm set 100 --hookscript none
```

### Test Hook

```bash
# Manually trigger
/root/script.sh 100 post-start
```

## Keywords

#hooks #events #automation #lifecycle

## Related Articles

- [[Automation]]
- [[Scripts]]
- [[Notifications]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
