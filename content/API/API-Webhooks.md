# Webhooks and Events Guide

## Overview

Webhooks allow Proxmox VE to send notifications to external services when events occur. Essential for automation and monitoring.

## Configure Webhooks

### Via Web UI

1. Navigate to **Datacenter → Permissions → API Tokens**
2. Create token for webhook service
3. Note token for webhook configuration

### Webhook Endpoint

```bash
# Example webhook URL
https://your-webhook-server.com/webhook/proxmox
```

## Event Types

### Supported Events

| Event | Description |
|-------|-------------|
| vm-start | VM started |
| vm-stop | VM stopped |
| vm-create | VM created |
| vm-delete | VM deleted |
| ct-start | Container started |
| ct-stop | Container stopped |
| backup-start | Backup started |
| backup-end | Backup completed |
| node-down | Node offline |
| cluster-join | Node joined cluster |

## Send Notifications

### Using curl

```bash
#!/bin/bash
# webhook-notify.sh

WEBHOOK_URL="https://hooks.example.com/webhook"
EVENT=$1
VMID=$2

PAYLOAD=$(cat <<EOF
{
  "event": "$EVENT",
  "vmid": $VMID,
  "timestamp": "$(date -Iseconds)",
  "node": "$(hostname)"
}
EOF
)

curl -X POST \
  -H "Content-Type: application/json" \
  -d "$PAYLOAD" \
  "$WEBHOOK_URL"
```

### Using pvesh

```bash
# Create event listener
pvesh create /nodes/localhost/events --event-type vm-start

# Or check logs
tail -f /var/log/pveproxy/access.log
```

## Automation Examples

### Slack Notification

```bash
#!/bin/bash
# slack-webhook.sh

SLACK_WEBHOOK="https://hooks.slack.com/services/xxx"
EVENT=$1
VMID=$2

PAYLOAD=$(cat <<EOF
{
  "text": "Proxmox Event: $EVENT on VM $VMID"
}
EOF
)

curl -X POST -H "Content-Type: application/json" \
  -d "$PAYLOAD" \
  "$SLACK_WEBHOOK"
```

### Discord Webhook

```json
{
  "content": "VM $VMID started",
  "embeds": [{
    "title": "Proxmox Notification",
    "color": 65280
  }]
}
```

## Monitor Events

### Watch Real-Time

```bash
# Monitor API access
tail -f /var/log/pveproxy/access.log

# Or using journal
journalctl -u pveproxy -f
```

## Keywords

#webhook #events #automation #notification #monitoring #slack

## Related Articles

- [[API]]
- [[Notifications]]
- [[Monitoring-Tools]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
