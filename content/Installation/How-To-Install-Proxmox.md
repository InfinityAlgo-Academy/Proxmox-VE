# How to Install Proxmox VE - Complete Beginner's Guide

## Question: How do I install Proxmox VE on my server?

### Answer: Follow this step-by-step guide

---

## Option 1: USB Installation (Recommended)

### Step 1: Download ISO
1. Go to proxmox.com/downloads
2. Download Proxmox VE ISO (latest version)
3. Download balenaEtcher or Rufus

### Step 2: Create Bootable USB
1. Open balenaEtcher
2. Select Proxmox ISO
3. Select USB drive
4. Click "Flash"

### Step 3: Boot from USB
1. Insert USB into server
2. Boot and press F2/F10/DEL for boot menu
3. Select USB as boot device

### Step 4: Installation Wizard
1. Select "Install Proxmox VE"
2. Choose target disk
3. Set country/timezone
4. Create password: admin123
5. Set hostname: pve1.yourdomain.com
6. Choose network (DHCP or static)
7. Wait for installation to complete

---

## Option 2: Network Installation (PXE)

### Step 1: Set up PXE Server
Install and configure DHCP/TFTP server on your network.

### Step 2: Configure
1. Point network boot to Proxmox iSCSI
2. Boot client from network
3. Follow installation wizard

---

## Option 3: Virtual Installation (Testing)

### Install in VirtualBox/VMware
```bash
# Minimum requirements
# CPU: 2 cores
# RAM: 4GB
# Disk: 32GB
```

---

## Keywords

#installation #how-to #beginner #usb #pxe

## Related Articles

- [[Proxmox-VE]]
- [[How-To-Initial-Configuration]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
