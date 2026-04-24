# CPU and Memory Performance Tuning

## Overview

Optimize Proxmox VE for maximum performance through CPU pinning, memory management, and NUMA configuration.

## CPU Pinning

### What is CPU Pinning

CPU pinning binds VM vCPUs to specific physical CPU cores, reducing context switching overhead.

### Pin VM to Cores

```bash
# Pin to cores 0-3
qm set 100 --cores 4 --cpuset 0-3

# Pin to cores 0,2,4,6 (even cores)
qm set 100 --cpuset 0,2,4,6

# Pin specific cores
qm set 100 --cpulimit 2 --cpuunits 1024
```

### Options Explained

| Option | Description |
|--------|-------------|
| --cores | Number of vCPUs |
| --cpuset | Physical cores to use |
| --cpulimit | CPU limit (%) |
| --cpuunits | CPU weight (0-1024) |

### Multi-Socket NUMA

```bash
# Enable NUMA
qm set 100 --numa 1

# Configure NUMA node
qm set 100 --numa0 memory=4096,cpus=0-3

# Second NUMA node
qm set 100 --numa1 memory=4096,cpus=4-7
```

## Huge Pages

### Enable Huge Pages

```bash
# Enable 2MB huge pages
qm set 100 --hugepages 1024

# Enable 1GB huge pages
qm set 100 --hugepages 1

# Check current huge pages
cat /proc/meminfo | grep -i huge
```

### Configure Host Huge Pages

```bash
# Allocate huge pages
echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages

# Check allocation
cat /proc/meminfo | grep -i huge

# Make persistent
echo "hugepages=1024" >> /etc/default/grub
update-grub
```

### Huge Page Sizes

| Size | Use Case |
|------|----------|
| 2MB | General workloads |
| 1GB | Large databases, HPC |

## Memory Optimization

### Memory Ballooning

```bash
# Disable balloon (for performance VMs)
qm set 100 --balloon 0

# Set minimum balloon
qm set 100 --balloon 4096

# Configure balloon drivers in guest
# Linux: apt install pciutils
# Windows: Install balloon driver
```

### Memory Locking

```bash
# Lock memory (prevent swap)
qm set 100 --lockmemory 1

# Check memory in guest
cat /sys/kernel/mm/transparent_hugepage/enabled
```

### NUMA Memory

```bash
# Enable NUMA
qm set 100 --numa 1

# Auto NUMA
qm set 100 --numa auto=1
```

## I/O Optimization

### I/O Thread

```bash
# Enable I/O thread
qm set 100 --iothread 1

# Add to VirtIO disk
qm set 100 --scsi0 local:,iothread=1,discard=1,ssd=1
```

### VirtIO Drivers

```bash
# Optimal VirtIO config
qm set 100 --scsi0 local:,cache=none,iothread=1

# Enable discard
qm set 100 --scsi0 local:,discard=1

# SSD emulation
qm set 100 --scsi0 local:,ssd=1
```

### Disk Options

| Option | Description |
|--------|-------------|
| cache=none | No host cache |
| cache=writeback | Write back |
| discard=1 | TRIM support |
| ssd=1 | SSD emulation |
| iothread=1 | I/O thread |

## Network Optimization

### VirtIO Multi-Queue

```bash
# Enable multi-queue (4 queues)
qm set 100 --net0 virtio,bridge=vmbr0,queues=4

# Update running VM
qm set 100 --net0 virtio,bridge=vmbr0, queues=8

# Maximum 16 queues
```

### Network Options

| Option | Description |
|--------|-------------|
| queues=N | Number of queues |
| rx=N | RX queues |
| tx=N | TX queues |

### Packet Steering

```bash
# RSS configuration in guest
ethtool -L eth0 combined 4
```

## CPU Mode Optimization

### Host CPU Mode

```bash
# Use host CPU (best performance)
qm set 100 --cpu host

# Enable flags
qm set 100 --cpu host --flags +vmx,+svm

# Specific CPU type
qm set 100 --cpu Penryn
qm set 100 --cpu Haswell-noTSX
```

### CPU Flags

| Flag | Description |
|------|-------------|
| +vmx | Intel VT-x |
| +svm | AMD-V |
| +aes | AES-NI |
| +avx | AVX |
| +avx2 | AVX2 |

## Real-Time Performance

### Enable Real-Time

```bash
# Configure real-time
qm set 100 --realtime 1

# Set CPU mask
qm set 100 --cpumask 0-3
```

### RT Kernel

```bash
# Install RT kernel
apt install proxmox-kernel-real-time

# Update GRUB
update-grub
reboot
```

## Benchmarking

### CPU Benchmark

```bash
# In VM: sysbench
sysbench cpu --cpu-max-prime=20000 run

# In VM: stress
stress --cpu 4
```

### Memory Benchmark

```bash
# In VM: sysbench
sysbench memory --memory-block-size=1M --memory-total-size=10G run
```

### Disk Benchmark

```bash
# In VM: fio
fio --name=seqread --ioengine=libaio --direct=1 --bs=4k --numjobs=1 --rw=read --size=1G --runtime=30 --filename=/tmp/test

# Or: dd
dd if=/dev/zero of=/tmp/test bs=4k count=250000 oflag=direct
```

## Monitoring

### Performance Metrics

```bash
# CPU usage
pvesh get /nodes/localhost/status

# VM stats
qm monitor 100

# Memory
qm exec 100 -- cat /proc/meminfo

# In guest commands
pct exec 100 -- cat /proc/meminfo
```

### Real-Time Monitoring

```bash
# Monitor VM
qm start 100

# Watch stats
watch -n1 "qm exec 100 -- cat /proc/stat"
```

## Quick Reference

### Optimization Checklist

```bash
# CPU
qm set 100 --cpu host --cores 4

# Memory
qm set 100 --balloon 0 --lockmemory 1

# NUMA
qm set 100 --numa 1

# I/O Thread
qm set 100 --iothread 1

# Network
qm set 100 --net0 virtio,bridge=vmbr0,queues=4
```

## Keywords

#performance #cpu-pinning #huge-pages #numa #optimization #io-thread #real-time

## Related Articles

- [[Advanced]]
- [[Nested-Virtualization]]
- [[SR-IOV]]
- [[Storage-Advanced]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
