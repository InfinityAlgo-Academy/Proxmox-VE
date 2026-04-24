# Boot Delay Configuration Guide

## Overview

Boot delay adds a pause before VM startup, ensuring dependencies are ready. Critical for VMs that depend on storage, network, or other services.

## Configure Boot Delay

### Per VM Delay

```bash
# Add 30 second delay
qm set 100 --startup-delay 30

# No delay
qm set 100 --startup-delay 0
```

### Global Delay Setting

```bash
# Set in datacenter.conf
pvesh put /cluster/options --startup-delay 10
```

## When to Use Boot Delays

### Storage Dependencies

```bash
# NFS storage VMs
qm set 100 --startup-delay 10

# Wait for NFS mount
# Make sure: /etc/pve/firewall/cluster.fw has rules for NFS
```

### Network Dependencies

```bash
# VMs needing network services
qm set 100 --startup-delay 20

# Wait for:
# - DHCP server
# - DNS
# - Active Directory
```

### Application Dependencies

```bash
# Web app depends on database
qm set 101 --startup-delay 30   # Database first
qm set 102 --startup-delay 40   # Web app after
```

## Calculate Delay

### Safe Formula

```
VM delay = (previous_VM_number * 10) + base_delay

Example:
VM 100: delay 0
VM 101: delay 10
VM 102: delay 20
```

## Test Boot Delays

### Monitor Startup

```bash
# Watch logs
journalctl -u pve-guest-manager -f

# Check VM status
qm status 100
```

## Related Articles

- [[Auto-Start]]
- [[Startup-Delays]]
- [[Storage-Configuration]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
