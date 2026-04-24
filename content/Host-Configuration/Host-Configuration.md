---
title: Host Configuration
---

# Host Configuration

## System Info

```bash
# Show host status
pvesh get /nodes/localhost/status

# Show CPU info
pvesh get /nodes/localhost/cpu

# Show memory
pvesh get /nodes/localhost/memory
```

## Host Settings

```bash
# Set hostname
hostnamectl set-hostname proxmox1

# Set timezone
timedatectl set-timezone America/New_York

# Update hosts file
echo "192.168.1.100 proxmox1" >> /etc/hosts
```

## See Also

- [[Host-Configuration]]
- [[Kernel-Configuration]]
- [[Installation]]
[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
