---
title: Cloud-Init
---

# Cloud-Init

## Overview

Cloud-Init is the industry standard method for initializing cloud instances.

## Use with Proxmox

```bash
# Add cloud-init to VM
qm set 100 --ide2 local:cloudinit,media=cdrom
```

## Configuration

```yaml
#cloud-config
hostname: myvm
username: admin
password: $6$rounds=4096$salt$hash
ssh_authorized_keys:
  - ssh-rsa AAAA...
packages:
  - curl
  - wget
runcmd:
  - curl https://example.com/init.sh | bash
```

## See Also

- [[Cloud]]
- [[VM]]
- [[Templates]]
[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
