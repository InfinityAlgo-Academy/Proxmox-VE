---
title: Bridge Config
---

# Bridge Configuration

## Network Bridge

```bash
# Create bridge
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bridge-ports eth0
    bridge-stp off
    bridge-fd 0
```

## Bridge Commands

| Command | Description |
|---------|-------------|
| `brctl show` | Show bridges |
| `brctl addbr vmbr1` | Add bridge |
| `brctl delbr vmbr1` | Delete bridge |

## See Also

- [[index|Back to Proxmox VE]]