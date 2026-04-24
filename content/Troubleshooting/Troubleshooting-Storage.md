---
title: Troubleshooting Storage
---

# Troubleshooting Storage

## Storage Not Available

### Check 1: Storage Status

```bash
# List storage
pvesm status

# Test storage
pvesm scan-lvm /dev/sda
```

### Check 2: Disk Health

```bash
# Check SMART
smartctl -a /dev/sda

# Test disk
badblocks -schu /dev/sda
```

### Check 3: LVM Issues

```bash
# Show LVM
pvscan
vgscan

# Fix LVM
vgchange -ay
```

## Low Disk Space

```bash
# Find large files
du -sh /var/lib/vz/*

# Clean old backups
rm -rf /var/lib/vz/dump/*
```

## See Also

- [[Troubleshooting]]
- [[Storage]]
- [[LVM-Management]]
[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
