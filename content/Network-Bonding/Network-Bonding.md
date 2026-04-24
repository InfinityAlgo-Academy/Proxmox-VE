---
title: Network Bonding
---

# Network Bonding

## Create Bond

```bash
# Add bond interface
auto bond0
iface bond0 inet static
    slaves eth0 eth1
    bond-mode 802.3ad
    bond-miimon 100
    bond-lacp-rate 1
```

## Bond Modes

| Mode | Description |
|-----|-------------|
| 0 | balance-rr |
| 1 | active-backup |
| 4 | 802.3ad (LACP) |

## See Also

- [[Networking]]
- [[VLAN-Config]]
- [[Network-Config]]
[[index|Back to Proxmox VE]]
