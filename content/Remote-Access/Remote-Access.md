---
title: Remote Access
---

# Remote Access

## VPN

### WireGuard

```bash
# Install WireGuard
apt install wireguard -y

# Generate keys
wg genkey | tee private.key
wg pubkey < private.key > public.key
```

### OpenVPN

```bash
# Install OpenVPN
apt install openvpn easy-rsa -y
```

## SSH

```bash
# Connect to Proxmox
ssh root@192.168.1.100

# SSH to VM
ssh user@192.168.1.101
```

## See Also

- [[VPN]]
- [[Security]]
- [[Remote-Access]]

[[index|Back to Proxmox VE]]
