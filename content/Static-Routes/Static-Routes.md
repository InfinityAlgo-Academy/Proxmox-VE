---
title: Static Routes
tags:
  - route
  - routing
  - network
  - ip
---

# Static Routes

## Overview

Static routes define specific paths for network traffic.

## Add Static Route

```bash
# Add route to network
ip route add 192.168.2.0/24 via 192.168.1.1

# Add route to host
ip route add 192.168.2.100 via 192.168.1.1

# Add default route
ip route add default via 192.168.1.1
```

## Persistent Routes

```bash
# /etc/network/interfaces
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    up ip route add 192.168.2.0/24 via 192.168.1.1
    down ip route del 192.168.2.0/24 via 192.168.1.1
```

## Show Routes

```bash
# Show routing table
ip route

# Show routing table (detailed)
ip route show

# Show all routes
route -n
```

## Delete Route

```bash
# Delete route
ip route del 192.168.2.0/24 via 192.168.1.1
```

## Troubleshooting

- No connectivity: Check route table
- Wrong path: Verify gateway

## See Also

- [[index|Back to Proxmox VE]]
- [[Network-Config]]
- [[Networking]]