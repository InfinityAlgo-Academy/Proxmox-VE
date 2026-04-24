# Proxmox VE Performance Guide

## Table of Contents

1. [CPU Performance](#cpu-performance)
2. [Memory Performance](#memory-performance)
3. [Storage Performance](#storage-performance)
4. [Network Performance](#network-performance)
5. [VM Optimization](#vm-optimization)
6. [Container Optimization](#container-optimization)

---

## CPU Performance

### CPU Modes

```bash
# Host passthrough
qm set <vmid> --cpu host

# Emulated
qm set <vmid> --cpu x86-64v2

# CPU pinning
qm set <vmid> --cores 2 --cpuset 0-1
```

### NUMA

```bash
# Enable NUMA
qm set <vmid> --numa 1

# Configure NUMA
qm set <vmid> --numa 1 --numa0 memory=4096,cpus=0-1
```

---

## Memory Performance

### Huge Pages

```bash
# Enable huge pages
qm set <vmid> --hugepages 1024

# Configure huge pages
echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```

### Memory Ballooning

```bash
# Enable balloon
qm set <vmid> --balloon 2048

# Disable balloon (performance critical)
qm set <vmid> --balloon 0
```

---

## Storage Performance

### I/O Thread

```bash
# Enable I/O thread
qm set <vmid> --scsi0 local:,iothread=1
```

### Cache Modes

```bash
# None (fastest)
qm set <vmid> --scsi0 local:,cache=none

# Write-through (safe)
qm set <vmid> --scsi0 local:,cache=writethrough

# Write-back (fast)
qm set <vmid> --scsi0 local:,cache=writeback
```

### Discard

```bash
# Enable discard
qm set <vmid> --scsi0 local:,discard=1,ssd=1
```

---

## Network Performance

### VirtIO

```bash
# Use VirtIO
qm set <vmid> --net0 virtio,bridge=vmbr0
```

### Multi-Queue

```bash
# Enable multi-queue
qm set <vmid> --net0 virtio,bridge=vmbr0,queues=4
```

### Interrupt Coalescing

```bash
# Tune network
ethtool -C eth0 rx-usecs 100 tx-usecs 100
```

---

## VM Optimization Checklist

- [ ] Use VirtIO drivers
- [ ] Enable I/O thread
- [ ] Enable multi-queue
- [ ] Disable balloon for critical VMs
- [ ] Use host CPU
- [ ] Enable huge pages
- [ ] Pin CPUs

---

## Keywords

#performance #optimization #tuning #cpu #memory #storage #network

## Links

- [[Proxmox-VE]] - Main documentation
- [[VM]] - VM management
- [[Advanced]] - Advanced configuration
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
