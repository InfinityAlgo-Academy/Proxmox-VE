---
title: Troubleshooting Boot Issues
---

# Troubleshooting Boot Issues

## VM Won't Boot

### Check 1: Boot Order

```bash
qm show_boot_info 100
# Verify boot device exists
qm start 100 --boot order=scsi0
```

### Check 2: Disk Full

```bash
# Check disk space
qm monitor 100 info block

# Resize disk
qm resize 100 scsi0 +10G
```

### Check 3: ISO Not Mounted

```bash
# Mount ISO
qm set 100 --ide2 local:iso/ubuntu.iso,media=cdrom
```

## Common Errors

| Error | Solution |
|-------|----------|
| No bootable device | Check boot order |
| Boot loop | Disable secure boot |
| PXE error | Check network boot |

## See Also

- [[Troubleshooting]]
- [[Boot-Configuration]]
- [[EFI-Configuration]]

[[index|Back to Proxmox VE]]
