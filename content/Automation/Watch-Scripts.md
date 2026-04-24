# Watch Scripts for Monitoring

## Overview

Watch scripts monitor system state and trigger actions based on changes. Essential for automated responses to VM events.

## Watch Basics

### Simple Monitor

```bash
#!/bin/bash
# monitor.sh - Monitor VM state

while true; do
    STATUS=$(qm status 100)
    if [ "$STATUS" != "running" ]; then
        echo "VM 100 stopped: $STATUS"
        # Send alert
    fi
    sleep 60
done
```

### Using pvesh

```bash
#!/bin/bash
# monitor-vms.sh - Watch VM states

watch_vm() {
    local vmid=$1
    qm status $vmid
}

while true; do
    for vm in 100 101 102; do
        status=$(watch_vm $vm)
        echo "VM $vm: $status"
    done
    sleep 30
done
```

## Event-Based Monitoring

### VM State Change

```bash
#!/bin/bash
# vm-monitor.sh - React to VM events

PREVIOUS_STATE=""

while true; do
    STATE=$(qm status 100)
    
    if [ "$STATE" != "$PREVIOUS_STATE" ]; then
        case $STATE in
            running)
                echo "VM started"
                # Action
                ;;
            stopped)
                echo "VM stopped"
                # Action
                ;;
        esac
        PREVIOUS_STATE=$STATE
    fi
    sleep 10
done
```

### Storage Monitoring

```bash
#!/bin/bash
# storage-monitor.sh - Check storage

THRESHOLD=90

while true; do
    for storage in $(pvesm status | grep active | awk '{print $1}'); do
        USAGE=$(pvesm status $storage | grep $storage | awk '{print int($5)}')
        
        if [ $USAGE -gt $THRESHOLD ]; then
            echo "WARNING: $storage at ${USAGE}%"
        fi
    done
    sleep 300
done
```

## Watch Commands

### Built-in Watch

```bash
# Watch VM metrics
qm monitor 100

# Watch resources
watch -n1 'qm status && pvesm status'
```

## Keywords

#monitoring #watch #automation #observability

## Related Articles

- [[Automation]]
- [[Scripts]]
- [[Monitoring-Tools]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
