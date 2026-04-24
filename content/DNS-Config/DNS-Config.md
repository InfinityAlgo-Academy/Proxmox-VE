---
title: DNS Configuration
---

# DNS Configuration

## Set DNS

```bash
# Set DNS for container
pct set 200 --dns 8.8.8.8

# Set DNS search domain
pct set 200 --dns-search local.domain
```

## DNS Server

```bash
# Install DNS server
apt install bind9 -y

# Configure
nano /etc/bind/named.conf
```

## See Also

- [[index|Back to Proxmox VE]]