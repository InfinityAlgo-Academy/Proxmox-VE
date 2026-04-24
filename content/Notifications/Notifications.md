# Notifications Guide

## Email

```bash
# Configure email
# Datacenter → Options → Email
```

## Telegram

```bash
# Create bot via @BotFather
# Send message via API
curl -s -X POST https://api.telegram.org/bot<token>/sendMessage -d chat_id=<chat_id> -d text="Alert"
```
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
