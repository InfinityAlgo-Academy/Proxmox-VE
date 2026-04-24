---
title: Email
---

# Email

## Email Server

```bash
# Install Postfix
apt install postfix -y

# Configure
nano /etc/postfix/main.cf
```

## Relay

```bash
# Configure relay
relayhost = smtp.example.com
smtp_sasl_auth_enable = yes
```

## See Also

- [[index|Back to Proxmox VE]]