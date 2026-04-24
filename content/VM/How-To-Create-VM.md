# How to Create a Virtual Machine - Complete Beginner's Guide

## Question: How do I create a VM in Proxmox VE?

### Answer: Follow these simple steps

---

## How to Create Using Web UI

### Step 1: Create New VM
1. Go to **Datacenter** → **Create VM**
2. Enter VM name: my-vm
3. Select OS: Linux/Debian (12)
4. CD/DVD: Select ISO

### Step 2: Configure Hardware
- **CPU**: 2 cores
- **Memory**: 4096 MB
- **Hard Disk**: 32 GB
- **Network**: VirtIO, Bridge: vmbr0

### Step 3: Finish
1. Click **Finish**
2. VM appears in list
3. Start VM and install OS

---

## How to Using CLI

```bash
qm create 100 --name my-vm --memory 4096 --cores 2 --scsi0 local:32 --net0 virtio,bridge=vmbr0
```

---

## Keywords

#vm #how-to #beginner #create

## Related Articles

- [[VM]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
