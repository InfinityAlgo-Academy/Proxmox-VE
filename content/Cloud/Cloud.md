---
title: Cloud
---

# Cloud Integration

## Proxmox Cloud Features

- Cloud-Init support
- Cloud-Init templates
- Network cloud configuration

## Deploy from Cloud Image

```bash
# Download cloud image
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img

# Create VM from cloud image
qm importdisk 100 jammy-server-cloudimg-amd64.img local-lvm

# Configure cloud-init
qm set 100 --ide2 local:cloudinit,media=cdrom
qm set 100 --sshkey ~/.ssh/id_rsa.pub
qm set 100 --ipconfig0 ip=dhcp
```

## Cloud-Init Examples

- [[Cloud-Init]]
- [[Cloud-Apps]]

## See Also

- [[Cloud-Init]]
- [[Templates]]
- [[VM]]

[[index|Back to Proxmox VE]]
