---
title: How to Configure Networking - Complete Beginner's Guide
---

# How to Configure Networking - Complete Beginner's Guide

## Question: How do I set up networking in Proxmox VE?

### Answer: Follow these steps

---

## How to Configure Basic Network

### Step 1: Access Network Settings
Go to **Datacenter** → **Network**

### Step 2: Create Bridge
1. Click **Create** → **Linux Bridge**
2. Name: vmbr0
3. Bridge ports: eth0
4. Click **Create**

### Step 3: Configure IP
```bash
# Via CLI
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
```

---

## How to Add VLAN

### Step-by-Step:
1. **Create** → **Linux Bridge**
2. Add VLAN tagged bridge: vmbr0.100
3. Set VLAN ID: 100
4. Save

---

## Keywords

#networking #how-to #bridge #vlan #beginner

## Related Articles

- [[Networking]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
