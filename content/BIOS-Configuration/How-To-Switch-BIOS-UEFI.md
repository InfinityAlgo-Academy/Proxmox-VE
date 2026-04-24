# How to Switch from BIOS to UEFI - Complete Guide

## Question: How do I switch my VM from BIOS to UEFI?

### Answer: Change BIOS type and configure boot properly

---

## Why Switch to UEFI?

### Benefits of UEFI
- **Secure Boot**: Protects from boot malware
- **GPT Support**: Disks larger than 2TB
- **Faster Boot**: Modern boot technology
- **Better Features**: NVRAM, restart fixes

---

## How to Change BIOS Type

### Method 1: Web UI
1. Go to VM → **Options**
2. Edit **BIOS**
3. Select **OVMF**
4. Save

### Method 2: CLI
```bash
qm set 100 --bios ovmf
```

---

## How to Add EFI Disk

### Create EFI Partition
```bash
# Add EFI disk
qm set 100 --efidisk0 local:1G
```

### Verify EFI Disk
```bash
qm config 100 | grep efidisk
```

---

## How to Configure Boot Order for UEFI

### Set Boot Order
```bash
# Boot from EFI disk first
qm set 100 --boot order=efidisk0

# Or with other options
qm set 100 --boot order=efidisk0;scsi0
```

---

## How to Enable Secure Boot

### Prerequisites
- OVMF BIOS enabled
- EFI disk added

### Enable Secure Boot
```bash
# Enable secure boot
qm set 100 --bios ovmf --secure-boot 1

# Default VM certificates
qm set 100 --bios ovmf
```

---

## Keywords

#uefi #secureboot #bios #conversion #how-to #efi

## Related Articles

- [[BIOS-Configuration]]
- [[EFI-Configuration]]
- [[VM]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
