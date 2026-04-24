---
title: How to Back Up Containers vs VMs - Complete Guide
---

# How to Back Up Containers vs VMs - Complete Guide

## Question: What's the difference in backing up LXC containers vs VMs?

### Answer: Similar process, some key differences

---

## Container (LXC) Backup

### How to Back Up Container
```bash
# Stop and backup
pct stop 200
vzdump --mode stop --vmid 200 --storage local

# Or live backup
vzdump --mode snapshot --vmid 200 --storage local
```

### Container-Specific Options
```bash
# Include config only
vzdump --mode stop --vmid 200 --storage local --notes-link 0
```

---

## VM (QEMU) Backup

### How to Back Up VM
```bash
# Live snapshot (no downtime)
vzdump --mode snapshot --vmid 100 --storage local

# Suspend mode (brief pause)
vzdump --mode suspend --vmid 100 --storage local

# Stop mode (full downtime)
vzdump --mode stop --vmid 100 --storage local
```

### VM-Specific Options
```bash
# Include cloud-init data
vzdump --mode snapshot --vmid 100 --storage local --cloud-init local:cloudinit
```

---

## Key Differences

| Feature | Container | VM |
|---------|----------|-----|
| Size | Smaller | Larger |
| Speed | Faster | Slower |
| Live backup | Yes | Yes |
| Mode: snapshot | Best | Good |
| Exclude CD-ROM | - | Yes |

---

## Keywords

#container-backup #vm-backup #lxc #how-to #differences

## Related Articles

- [[Backup]]
- [[Containers]]
- [[VM]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
