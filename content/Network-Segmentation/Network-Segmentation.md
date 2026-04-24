---
title: Network Segmentation
---

# Network Segmentation

## VLANs

```bash
# Create VLAN
ip link add link vmbr0 name vmbr0.100 type vlan id 100

# Configure VLAN
ip addr add 192.168.100.1/24 dev vmbr0.100
ip link set vmbr0.100 up
```

## Network Isolation

```bash
# Separate networks
auto vmbr1
iface vmbr1 inet static
    address 10.0.0.1
    netmask 255.255.255.0
    bridge-ports none
```

## See Also

- [[index|Back to Proxmox VE]]