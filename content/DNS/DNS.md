---
title: DNS
---

# DNS

## Configure DNS

```bash
# Edit /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4

# Test DNS
nslookup proxmox.local
dig proxmox.local
```

## DNS for VMs

```bash
# Set DNS for container
pct set 200 --dns 8.8.8.8
pct set 200 --dns-search local.domain
```

## See Also

- [[Network]]
- [[DNS-Config]]
[[index|Back to Proxmox VE]]
