# Nested Virtualization Complete Guide

## Overview

Nested virtualization allows you to run a hypervisor inside a virtual machine. This enables running Proxmox VE inside Proxmox VE for testing, development, and learning environments.

## Why Use Nested Virtualization

### Use Cases

- **Testing**: Test new Proxmox versions without affecting production
- **Learning**: Practice cluster setup and migrations
- **Development**: Build and test automation scripts
- **Demos**: Showcase Proxmox features to others
- **Multi-tenant**: Isolated test environments

### Limitations

- Performance overhead (~10-20% slower)
- Cannot nest Hyper-V/ESXi in production cluster
- Limited hardware passthrough

## Prerequisites

### Hardware Requirements

- CPU with virtualization extensions (VT-x/AMD-V)
- Minimum 8 cores recommended
- Minimum 16GB RAM
- 100GB+ storage

### Software Requirements

- Proxmox VE 8.0+
- Nested KVM module

## Enable Nested Virtualization

### Intel Processors

```bash
# Check current status
cat /sys/module/kvm_intel/parameters/nested

# Enable nested virtualization
echo "options kvm_intel nested=1" | tee /etc/modprobe.d/kvm-intel.conf

# Reload module
modprobe -r kvm_intel
modprobe kvm_intel

# Verify
cat /sys/module/kvm_intel/parameters/nested
# Output: Y (enabled)
```

### AMD Processors

```bash
# Check current status
cat /sys/module/kvm_amd/parameters/nested

# Enable nested virtualization
echo "options kvm_amd nested=1" | tee /etc/modprobe.d/kvm-amd.conf

# Reload module
modprobe -r kvm_amd
modprobe kvm_amd

# Verify
cat /sys/module/kvm_amd/parameters/nested
# Output: Y (enabled)
```

### Persistent Configuration

```bash
# Create configuration file
nano /etc/modprobe.d/kvm.conf

# For Intel:
options kvm_intel nested=1

# For AMD:
options kvm_amd nested=1

# For both (conditional):
options kvm_intel nested=1
options kvm_amd nested=1
```

## Create Nested Proxmox VM

### VM Configuration

```bash
# Create nested Proxmox VM
qm create 9000 \
  --name "nested-proxmox" \
  --memory 16384 \
  --cores 4 \
  --cpu host \
  --numa 1 \
  --net0 virtio,bridge=vmbr0 \
  --scsi0 local-lvm:32,discard=1,ssd=1 \
  --bootdisk scsi0 \
  --bios ovmf \
  --cpu host \
  --flags +vmx

# Enable nested in VM config
qm set 9000 --cpu host --flags +vmx

# For AMD, use +svm instead
qm set 9000 --cpu host --flags +svm
```

### Optimal VM Settings

| Setting | Value | Reason |
|---------|-------|--------|
| CPU | host | Full virtualization |
| Cores | 4+ | Performance |
| Memory | 8GB+ | Proxmox needs |
| Disk | 64GB+ | Storage |
| Network | VirtIO | Performance |

### Install Proxmox in VM

```bash
# Mount ISO
qm set 9000 --ide2 local:iso/proxmox-8.0.iso,media=cdrom

# Start VM
qm start 9000

# Access console
qm console 9000

# Complete installation via web UI:
# https://your-pve:8006
```

## Configure LXC for Nested

### Enable Nesting in Containers

```bash
# Enable basic nesting
pct set 100 --features nesting=1

# Enable full nesting (Docker/Podman)
pct set 100 --features nesting=1,keyctl=1,fuse=1,shiftfs=1

# Create container with nesting
pct create 200 \
  local:vztmpl/debian-12.tar.gz \
  --rootfs local:10 \
  --features nesting=1,keyctl=1,fuse=1
```

### Nested Container Examples

```bash
# Docker in container
pct create 201 local:vztmpl/debian-12.tar.gz \
  --rootfs local:20 \
  --features nesting=1,keyctl=1,fuse=1

pct start 201
pct exec 201 apt update
pct exec 201 apt install -y docker.io

# Podman in container
pct create 202 local:vztmpl/ubuntu-22.04.tar.gz \
  --rootfs local:20 \
  --features nesting=1,keyctl=1,fuse=1

pct start 202
pct exec 202 apt update
pct exec 202 apt install -y podman
```

### Required Container Features

| Feature | Purpose |
|---------|---------|
| `nesting=1` | Allow nested VMs |
| `keyctl=1` | Keyring access |
| `fuse=1` | FUSE filesystem |
| `shiftfs=1` | ID shifting |

## Performance Optimization

### CPU Configuration

```bash
# Use host CPU for best performance
qm set 9000 --cpu host

# Pin to specific cores
qm set 9000 --cpuset 0-3

# Enable NUMA
qm set 9000 --numa 1
```

### Memory Optimization

```bash
# Lock memory (prevent swap)
qm set 9000 --lockmemory 1

# Disable balloon
qm set 9000 --balloon 0
```

### Network Optimization

```bash
# Use VirtIO with multi-queue
qm set 9000 --net0 virtio,bridge=vmbr0,queues=4
```

## Nested Cluster

### Create Test Cluster

```bash
# On first node
pvecm create node1

# On second node
pvecm add 192.168.1.101

# On third node
pvecm add 192.168.1.102
```

### Test HA

```bash
# Enable HA for test VMs
ha-manager enable vm:9000

# Configure HA group
ha-manager group add test-cluster pve1 pve2 pve3
```

## Troubleshooting

### Nested Not Working

```bash
# Check KVM module
lsmod | grep kvm

# Verify CPU supports VT-x/AMD-V
egrep -c 'vmx|svm' /proc/cpuinfo

# Check VM config
qm config 9000
```

### Performance Issues

```bash
# Check CPU usage in guest
qm exec 9000 -- cat /proc/cpuinfo

# Monitor host resources
top
htop

# Check for CPU throttling
qm config 9000
# Ensure no CPU limits set
```

## Best Practices

1. **Resource allocation**: Give nested VMs adequate resources
2. **Use VirtIO**: Best performance
3. **Separate networks**: Use isolated VM bridge
4. **Limit nesting**: One level deep is optimal
5. **Monitor performance**: Check resource usage

## Use Cases

### CI/CD Pipelines

```yaml
# GitHub Actions example
- name: Test Proxmox deployment
  uses: proxmox/action/deploy@v1
  with:
    host: ${{ secrets.PVE_HOST }}
    vm-id: 9000
    template: ubuntu-22.04
```

### Training Environments

```bash
# Create training cluster
for i in 9001 9002 9003; do
  qm create $i --name "training-$i" \
    --memory 8192 --cores 4 \
    --cpu host --net0 virtio,bridge=vmbr0
done
```

## Keywords

#nested #nested-virtualization #kvm #hypervisor #test-environment #development #training #learning-lab

## Related Articles

- [[Advanced]]
- [[VM]]
- [[Performance]]
- [[Cluster]]