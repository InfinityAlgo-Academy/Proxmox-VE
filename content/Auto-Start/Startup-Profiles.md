# Startup Profiles and Templates

## Overview

Startup profiles store pre-configured boot settings for quick deployment. Ideal for standardizing VM configurations.

## Create Startup Profile

### Save Current Config

```bash
# Export VM config
qm config 100 > /root/vm-startup-profile.txt

# Or use JSON
qm config 100 --output-format json
```

### Template Variables

```bash
# Create template file
# File: /root/templates/web-server.xml
<vm>
  <memory>4096</memory>
  <cores>2</cores>
  <boot>order=scsi0</boot>
</vm>
```

## Use Templates

### Clone with Template

```bash
# Use template on clone
qm clone 100 101 --name "new-vm" --full 1
```

### Bulk VM Creation

```bash
# Create multiple VMs
for i in 110 111 112 113 114; do
  qm create $i \
    --name "web-$i" \
    --memory 4096 \
    --cores 2 \
    --scsi0 local:32 \
    --net0 virtio,bridge=vmbr0 \
    --onboot 1
done
```

## Standard Profiles

### Minimum Profile

```bash
# Basic VM
qm set 100 \
  --memory 2048 \
  --cores 2 \
  --boot order=scsi0
```

### Database Profile

```bash
# Database VM
qm set 101 \
  --memory 8192 \
  --cores 4 \
  --scsi0 local:64 \
  --boot order=scsi0 \
  --onboot 1
```

### Web Server Profile

```bash
# Web VM
qm set 102 \
  --memory 4096 \
  --cores 2 \
  --scsi0 local:32 \
  --net0 virtio,bridge=vmbr0 \
  --boot order=scsi0 \
  --onboot 1
```

## Automation

### Script Template

```bash
#!/bin/bash
# create-vm-from-template.sh

VMID=$1
NAME=$2
MEMORY=${3:-4096}
CORES=${4:-2}

qm create $VMID \
  --name "$NAME" \
  --memory $MEMORY \
  --cores $CORES \
  --scsi0 local:32 \
  --net0 virtio,bridge=vmbr0 \
  --onboot 1 \
  --boot order=scsi0

echo "VM $VMID created"
```

### Usage

```bash
# Create VM
./create-vm-from-template.sh 900 web-server 4096 2

# Quick VM
./create-vm-from-template.sh 901 test-vm
```

## Related Articles

- [[Auto-Start]]
- [[Auto-Start-Configuration]]
- [[Templates]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
