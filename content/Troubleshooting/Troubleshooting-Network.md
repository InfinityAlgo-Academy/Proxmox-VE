---
title: Troubleshooting Network
---

# Troubleshooting Network

## No Network Connection

### Check 1: Bridge Exists

```bash
# Show bridges
brctl show

# Check IP
ip addr show vmbr0
```

### Check 2: VM Network Config

```bash
# Check VM network
qm set 100 --net0 virtio,bridge=vmbr0

# Restart networking
systemctl restart networking
```

### Check 3: Firewall

```bash
# Check firewall
ufw status

# Allow all traffic (test)
ufw disable
```

## DNS Issues

```bash
# Test DNS
nslookup proxmox.local 8.8.8.8

# Update resolv.conf
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

## See Also

- [[Troubleshooting]]
- [[Networking]]
- [[VLAN-Config]]
[[index|Back to Proxmox VE]]
