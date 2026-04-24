# How to Configure Legacy Boot - Complete Guide

## Question: How do I use legacy BIOS boot mode?

### Answer: Configure SeaBIOS for legacy boot

---

## When to Use Legacy Boot

### Use Cases
- Old Windows (XP, 7)
- MS-DOS
- Compatibility issues
- Testing legacy systems

---

## How to Enable Legacy Boot

### Set SeaBIOS
```bash
# Enable legacy BIOS
qm set 100 --bios seabios
```

---

## How to Configure Legacy Settings

### Disable UEFI Features
```bash
# Remove EFI disk
qm set 100 --delete efidisk0

# Use BIOS boot
qm set 100 --boot order=scsi0
```

---

## How to Add Boot Devices

### IDE Disks (Legacy)
```bash
# Add IDE disk
qm set 100 --ide0 local:32

# Set boot
qm set 100 --boot order=ide0
```

### SCSI Disks
```bash
# Add SCSI disk
qm set 100 --scsi0 local:32

# Boot from SCSI
qm set 100 --boot order=scsi0
```

---

## How Legacy Boot Works

### Boot Process
```
VM Starts → BIOS POST → Boot Order Check → Boot Device Found → Load OS
```

### Legacy vs UEFI
| Feature | Legacy BIOS | UEFI |
|---------|-----------|------|
| Boot Method | MBR | GPT |
| Drivers | In OS | Pre-OS |
| Startup | BIOS | UEFI |
| Size Limit | 2TB | 256TB+ |

---

## Keywords

#legacy #seabios #mbr #boot #how-to

## Related Articles

- [[BIOS-Configuration]]
- [[Boot-Configuration]]
- [[VM]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
