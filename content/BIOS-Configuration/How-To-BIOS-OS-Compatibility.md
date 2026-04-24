# How to Use BIOS with Different Operating Systems - Complete Guide

## Question: Which BIOS settings for different OS?

### Answer: OS-specific BIOS configuration

---

## Windows BIOS Settings

### Windows 10/11
```bash
# Use UEFI
qm set 100 --bios ovmf

# Add EFI disk
qm set 100 --efidisk0 local:1G

# Secure boot (optional)
qm set 100 --secureboot 1
```

### Windows 7 (Legacy)
```bash
# Use SeaBIOS
qm set 100 --bios seabios

# IDE disk
qm set 100 --ide0 local:32
```

### Windows XP (Legacy)
```bash
# Use SeaBIOS
qm set 100 --bios seabios

# IDE required
qm set 100 --ide0 local:20
```

---

## Linux BIOS Settings

### Ubuntu 20.04+
```bash
# UEFI recommended
qm set 100 --bios ovmf
qm set 100 --efidisk0 local:1G

# Secure boot enabled by default
```

### CentOS/RHEL
```bash
# UEFI
qm set 100 --bios ovmf
qm set 100 --efidisk0 local:1G
```

### Old Linux (Pre-UEFI)
```bash
# Use SeaBIOS
qm set 100 --bios seabios
```

---

## macOS BIOS Settings

### macOS (Hackintosh)
```bash
# OVMF required
qm set 100 --bios ovmf

# EFI disk
qm set 100 --efidisk0 local:1G
```

---

## Keywords

#os #windows #linux #macos #compatibility #how-to

## Related Articles

- [[BIOS-Configuration]]
- [[VM]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
