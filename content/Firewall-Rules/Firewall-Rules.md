---
title: Firewall Rules
tags:
  - firewall
  - security
  - iptables
  - network
---

# Firewall Rules

## Proxmox Firewall

### Enable Firewall

```bash
# Enable node firewall
pvenode modify --firewall 1
```

### Basic Rules

```bash
# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow Proxmox
iptables -A INPUT -p tcp --dport 8006 -j ACCEPT

# Allow API
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT
```

## VM Firewall

```bash
# Enable VM firewall
qm set 100 --net0 virtio,bridge=vmbr0,firewall=1

# Add rule via API
pvesh create /nodes/localhost/firewall/rules -d '{"action":"ACCEPT","protocol":"tcp","dport":80}'
```

## UFW (Simpler)

```bash
# Install
apt install ufw -y

# Default policy
ufw default deny incoming
ufw default allow outgoing

# Allow services
ufw allow 8006/tcp comment 'Proxmox Web'
ufw allow 22/tcp comment 'SSH'

# Enable
ufw enable
```

## Common Ports

| Port | Service |
|------|---------|
| 22 | SSH |
| 8006 | Proxmox Web |
| 3128 | SPICE |
| 5900-6005 | Console |

## See Also

- [[index|Back to Proxmox VE]]
- [[Security]]
- [[Network-Monitoring]]