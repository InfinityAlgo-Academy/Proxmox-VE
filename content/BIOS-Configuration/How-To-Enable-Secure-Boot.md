---
title: How to Enable Secure Boot - Complete Guide
---

# How to Enable Secure Boot - Complete Guide

## Question: How do I enable secure boot in Proxmox VMs?

### Answer: UEFI with secure boot configuration

---

## What is Secure Boot?

### Purpose
Secure boot ensures only signed operating systems can boot. This protects against:
- Boot malware
- Unauthorized OS loading
- Rootkits

---

## Prerequisites

### Requirements
- UEFI BIOS (OVMF)
- EFI disk
- Signed boot loader

---

## How to Enable Secure Boot

### Step 1: Ensure UEFI
```bash
# Check current BIOS
qm config 100 | grep bios

# Enable UEFI if needed
qm set 100 --bios ovmf
```

### Step 2: Add EFI Disk
```bash
# Create EFI disk
qm set 100 --efidisk0 local:1G
```

### Step 3: Enable Secure Boot
```bash
# Secure boot (default certificate)
qm set 100 --secureboot 1
```

### Alternative: Custom Keys
```bash
# Use different PK/KEK/DB
qm set 100 --bios ovmf --secure-boot custom
```

---

## How to Disable Secure Boot

### Disable Secure Boot
```bash
# Disable secure boot
qm set 100 --secureboot 0
```

---

## How to Verify Secure Boot

### Check Status
```bash
qm config 100 | grep -i secure
```

---

## Operating Systems Supporting Secure Boot

| OS | Secure Boot Support |
|----|-------------------|
| Windows 10/11 | Yes |
| Ubuntu 20.04+ | Yes |
| Fedora 33+ | Yes |
| Linux 5.4+ | Yes |

---

## Keywords

#secureboot #uefi #security #how-to #bios

## Related Articles

- [[BIOS-Configuration]]
- [[EFI-Configuration]]
- [[Security]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
