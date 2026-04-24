# How to Create a Container (LXC) - Complete Beginner's Guide

## Question: How do I create a container in Proxmox VE?

### Answer: Follow these simple steps

---

## How to Create Using Web UI

### Step 1: Create Container
1. Go to **Datacenter** → **Create CT**
2. Enter hostname: my-container
3. Select template: Debian-12

### Step 2: Configure
- **Root Disk**: 8 GB
- **CPU**: 2 cores
- **Memory**: 2048 MB
- **Network**: DHCP

### Step 3: Finish
1. Click **Finish**
2. Start container

---

## How to Using CLI

```bash
pct create 200 local:vztmpl/debian-12.tar.gz --rootfs local:8 --memory 2048 --cores 2
```

---

## Keywords

#container #lxc #how-to #beginner

## Related Articles

- [[Containers]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
