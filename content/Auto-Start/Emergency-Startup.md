# Emergency Startup Configuration

## Overview

Emergency startup configuration ensures critical VMs boot first during host startup, even before non-critical services. Essential for infrastructure VMs like DNS, DHCP, and authentication.

## Priority Startup

### Set Startup Order

```bash
# Infrastructure VMs first (lowest delay)
qm set 100 --startup-delay 0    # DNS/DHCP
qm set 101 --startup-delay 10   # Auth
qm set 102 --startup-delay 20   # File server
qm set 103 --startup-delay 30   # Web apps
```

### Group Priority

```bash
# Create startup groups
# File: /etc/pve/datacenter.conf
startup: 
  - vm: 100
    delay: 0
  - vm: 101
    delay: 10
```

## Critical VM Identification

### Tag Critical VMs

```bash
# Enable boot for all critical VMs
for vm in 100 101 102; do
  qm set $vm --onboot 1
done
```

## Startup Group Configuration

### Define Groups

```bash
# Infrastructure
ha-manager group add infra pve1 pve2 pve3

# Set priority
ha-manager crm add infra --vm 100 --priority 100
```

## Related Articles

- [[Auto-Start]]
- [[High-Availability]]
- [[Cluster]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
