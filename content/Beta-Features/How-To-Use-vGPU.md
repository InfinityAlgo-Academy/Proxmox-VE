# How to Use vGPU Features - Complete Guide

## Question: How do I use vGPU beta features?

### Answer: GPU virtualization options

---

## How to Enable vGPU

### Step 1: Check Support
```bash
# Check GPU
lspci | grep -i vga

# Check IOMMU
dmesg | grep -e DMAR -e IOMMU
```

---

## How to Configure Intel vGPU

### Step 1: Enable GVT-g
```bash
echo "i915.enable_gvt=1" >> /etc/modprobe.d/gvt.conf

# Reboot
reboot
```

### Step 2: Create vGPU
```bash
# List available
ls /sys/class/mdev_bus/

# Create vGPU
echo "i915-GVTg_V5_4" > /sys/class/mdev_bus/mdev0/device/mdev_supported_types/i915-GVTg_V5_4/create
```

---

## How to Assign vGPU to VM

### Step 1: Configure VM
```bash
qm set 100 --hostpci0 00:02.0,pcie=1,mdev=i915-GVTg_V5_4
```

---

## Keywords

#vgpu #gpu #intel-gvt #how-to

## Related Articles

- [[Beta-Features]]
- [[vGPU-Configuration]]
- [[Proxmox-VE]]