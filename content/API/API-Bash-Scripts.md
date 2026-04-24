# API Bash Scripts Guide

## Overview

Learn how to use bash scripts with Proxmox VE REST API for automation and command-line integration.

## Using curl

### Basic GET Request

```bash
#!/bin/bash

# Get cluster status
curl -s -k \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  https://pve.example.com:8006/api2/json/cluster/status
```

### Using Basic Auth

```bash
#!/bin/bash

# Get status with basic auth
curl -s -k -u "root@pam:password" \
  https://pve.example.com:8006/api2/json/cluster/status
```

### Pretty Output

```bash
#!/bin/bash

# jq for pretty output
curl -s -k -u "root@pam:password" \
  https://pve.example.com:8006/api2/json/cluster/status | jq '.data[]'
```

## VM Management Scripts

### Start All VMs

```bash
#!/bin/bash
# start-all-vms.sh

TOKEN="your-token-here"
PVE="pve.example.com"

VM_LIST=$(curl -s -k -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  https://$PVE:8006/api2/json/cluster/resources | \
  jq -r '.data[] | select(.type == "qemu") | .vmid')

for VMID in $VM_LIST; do
  echo "Starting VM $VMID..."
  curl -s -k -X POST -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
    https://$PVE:8006/api2/json/nodes/pve/qemu/$VMID/status/start
done
```

### Stop All VMs

```bash
#!/bin/bash
# stop-all-vms.sh

TOKEN="your-token-here"
PVE="pve.example.com"

VM_LIST=$(curl -s -k -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  https://$PVE:8006/api2/json/cluster/resources | \
  jq -r '.data[] | select(.type == "qemu") | .vmid')

for VMID in $VM_LIST; do
  echo "Stopping VM $VMID..."
  curl -s -k -X POST -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
    https://$PVE:8006/api2/json/nodes/pve/qemu/$VMID/status/stop
done
```

### Create VM

```bash
#!/bin/bash
# create-vm.sh

VMID=$1
NAME=$2
MEMORY=$3
CORES=$4

TOKEN="your-token-here"
PVE="pve.example.com"

curl -s -k -X POST \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"vmid\": $VMID,
    \"name\": \"$NAME\",
    \"memory\": $MEMORY,
    \"cores\": $CORES,
    \"ostype\": \"l26\",
    \"scsi0\": \"local:32\",
    \"net0\": \"virtio,bridge=vmbr0\"
  }" \
  https://$PVE:8006/api2/json/nodes/pve/qemu
```

### Clone VM

```bash
#!/bin/bash
# clone-vm.sh

SOURCE_VM=$1
TARGET_VM=$2
TARGET_NAME=$3

TOKEN="your-token-here"
PVE="pve.example.com"

curl -s -k -X POST \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"newid\": $TARGET_VM,
    \"name\": \"$TARGET_NAME\",
    \"full\": 1
  }" \
  https://$PVE:8006/api2/json/nodes/pve/qemu/$SOURCE_VM/clone
```

## Container Scripts

### Create Container

```bash
#!/bin/bash
# create-ct.sh

VMID=$1
TEMPLATE=$2
MEMORY=$3
CORES=$4
IP=$5

TOKEN="your-token-here"
PVE="pve.example.com"

curl -s -k -X POST \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"vmid\": $VMID,
    \"ostype\": \"debian\",
    \"rootfs\": \"local:8\",
    \"memory\": $MEMORY,
    \"cores\": $CORES,
    \"net0\": \"name=eth0,bridge=vmbr0,ip=$IP\"
  }" \
  https://$PVE:8006/api2/json/nodes/pve/lxc
```

### Execute Command

```bash
#!/bin/bash
# exec-ct.sh

VMID=$1
CMD="$2"

TOKEN="your-token-here"
PVE="pve.example.com"

curl -s -k -X POST \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"command\": \"$CMD\"
  }" \
  https://$PVE:8006/api2/json/nodes/pve/lxc/$VMID/status
```

## Backup Scripts

### Backup VM

```bash
#!/bin/bash
# backup-vm.sh

VMID=$1
STORAGE=${2:-local}
MODE=${3:-snapshot}

TOKEN="your-token-here"
PVE="pve.example.com"

DATE=$(date +%Y%m%d-%H%M)

curl -s -k -X POST \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"vmid\": $VMID,
    \"storage\": \"$STORAGE\",
    \"mode\": \"$MODE\"
  }" \
  https://$PVE:8006/api2/json/nodes/pve/vzdump
```

### List Backups

```bash
#!/bin/bash
# list-backups.sh

STORAGE=$1

TOKEN="your-token-here"
PVE="pve.example.com"

curl -s -k \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  https://$PVE:8006/api2/json/storage/$STORAGE/content | \
  jq -r '.data[] | select(.content == "backup") | .volid + " " + .size'
```

## User Management Scripts

### Create User

```bash
#!/bin/bash
# create-user.sh

USER=$1
PASSWORD=$2

TOKEN="your-token-here"
PVE="pve.example.com"

curl -s -k -X POST \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"userid\": \"$USER\",
    \"password\": \"$PASSWORD\"
  }" \
  https://$PVE:8006/api2/json/access/users
```

### Create API Token

```bash
#!/bin/bash
# create-token.sh

USER=$1
TOKEN_NAME=$2

TOKEN="your-token-here"
PVE="pve.example.com"

curl -s -k -X POST \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"tokenid\": \"$TOKEN_NAME\"
  }" \
  https://$PVE:8006/api2/json/access/users/$USER/tokens
```

## Storage Scripts

### Add Storage

```bash
#!/bin/bash
# add-storage.sh

STORAGE_ID=$1
STORAGE_TYPE=$2
PATH=$3

TOKEN="your-token-here"
PVE="pve.example.com"

curl -s -k -X POST \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"storage\": \"$STORAGE_ID\",
    \"type\": \"$STORAGE_TYPE\",
    \"path\": \"$PATH\",
    \"content\": \"images,rootdir\",
    \"is-rootdir\": 1
  }" \
  https://$PVE:8006/api2/json/storage
```

### Get Storage Usage

```bash
#!/bin/bash
# storage-usage.sh

TOKEN="your-token-here"
PVE="pve.example.com"

curl -s -k \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  https://$PVE:8006/api2/json/storage | \
  jq -r '.data[] | "\(.storage) \(.type) \(.avail)/\(.total)"'
```

## Monitoring Scripts

### Node Status

```bash
#!/bin/bash
# node-status.sh

TOKEN="your-token-here"
PVE="pve.example.com"

curl -s -k \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  https://$PVE:8006/api2/json/nodes | \
  jq -r '.data[] | "\(.node) - CPU:\(.cpu) MEM:\(.memtotal / 1024 / 1024 / 1024)GB"'
```

### VM List with Status

```bash
#!/bin/bash
# vm-list.sh

TOKEN="your-token-here"
PVE="pve.example.com"

curl -s -k \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  https://$PVE:8006/api2/json/cluster/resources | \
  jq -r '.data[] | select(.type == "qemu") | "\(.vmid) \(.status) \(.name)"'
```

## Utility Scripts

### Get API Version

```bash
#!/bin/bash
# api-version.sh

curl -s -k \
  https://pve.example.com:8006/api2/json/version | \
  jq '.data'
```

### Health Check

```bash
#!/bin/bash
# health-check.sh

TOKEN="your-token-here"
PVE="pve.example.com"

STATUS=$(curl -s -k \
  -H "Authorization: PVEAPIToken=root@pam=$TOKEN" \
  https://$PVE:8006/api2/json/cluster/status | \
  jq -r '.data[0].quorate')

if [ "$STATUS" = "true" ]; then
  echo "Cluster OK"
else
  echo "Cluster DOWN"
  exit 1
fi
```

## Keywords

#bash #curl #scripting #automation #cli

## Related Articles

- [[API]]
- [[API-Authentication]]
- [[API-Endpoints]]