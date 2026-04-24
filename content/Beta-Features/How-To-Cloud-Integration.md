# How to Use Cloud Integration Features - Complete Guide

## Question: How do I use cloud integration betas?

### Answer: Available cloud features

---

## How to Enable Cloud API

### Step 1: Enable Access
```bash
# Enable cloud API
pvesh put /access/options --api-enabled 1
```

---

## How to Use Cloud-Init Templates

### Step 1: Download Cloud Image
```bash
# Download Ubuntu cloud image
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img

# Import as template
qm import-img jammy-server-cloudimg-amd64.img local --format qcow2
```

### Step 2: Configure Cloud-Init
```bash
# Add cloud-init disk
qm set 100 --cloudinit local:cloudinit

# Configure user data
qm set 100 --ciuser admin --cipassword changeme
```

---

## Keywords

#cloud #cloud-init #integration #how-to

## Related Articles

- [[Beta-Features]]
- [[Cloud-Init]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
