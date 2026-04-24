---
title: Optimization
---

# Performance Optimization

## CPU Optimization

```bash
# Set CPU type
qm set 100 --cpu host

# Enable NUMA
qm set 100 --numa 1
```

## Memory Optimization

```bash
# Set memory
qm set 100 --memory 4096

# Enable memory balloon
qm set 100 --balloon 2048
```

## Network Optimization

```bash
# Enable virtio multi-queue
qm set 100 --net0 virtio,bridge=vmbr0,queues=4
```

## See Also

- [[index|Back to Proxmox VE]]