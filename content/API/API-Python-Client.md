# Python API Client Guide

## Overview

Learn how to use Python to interact with Proxmox VE API for automation and scripting.

## Setup

### Install Library

```bash
pip install proxmoxer
```

### Using pipenv

```bash
pipenv install proxmoxer
```

## Basic Usage

### Connect to API

```python
import proxmoxer

proxmox = proxmoxer.ProxmoxAPI(
    'https://pve.example.com:8006',
    user='root@pam',
    password='your_password',
    verify_ssl=False
)
```

### Using Token

```python
proxmox = proxmoxer.ProxmoxAPI(
    'https://pve.example.com:8006',
    user='root@pam',
    token_password='your_token',
    verify_ssl=False
)
```

## Cluster Operations

### Get Cluster Status

```python
# Get all cluster status
status = proxmox.cluster.status.get()

for node in status:
    print(f"{node['node']}: {node['status']}")
```

### Get Cluster Resources

```python
# Get all resources
resources = proxmox.cluster.resources.get()

# Filter by type
vms = [r for r in resources if r['type'] == 'qemu']
containers = [r for r in resources if r['type'] == 'lxc']
nodes = [r for r in resources if r['type'] == 'node']
```

## VM Operations

### List VMs

```python
nodes = proxmox.cluster.nodes.get()

for node in nodes:
    node_name = node['node']
    qemu_list = proxmox.nodes(node_name).qemu.get()
    
    for vm in qemu_list:
        print(f"VM {vm['vmid']}: {vm['name']}")
```

### Create VM

```python
vm_config = {
    'vmid': 900,
    'name': 'my-new-vm',
    'memory': 4096,
    'cores': 2,
    'ostype': 'l26',
    'scsi0': 'local:32',
    'net0': 'virtio,bridge=vmbr0',
    'boot': 'order=scsi0'
}

proxmox.nodes('pve1').qemu.post(**vm_config)
```

### Start VM

```python
proxmox.nodes('pve1').qemu(900).status.start.post()
```

### Stop VM

```python
# Graceful shutdown
proxmox.nodes('pve1').qemu(900).status.stop.post()

# Force stop
proxmox.nodes('pve1').qemu(900).status.stop.post(force=1)
```

### Get VM Status

```python
vm_status = proxmox.nodes('pve1').qemu(900).status.get()
print(vm_status['status'])
```

### Update VM Config

```python
# Add more memory
proxmox.nodes('pve1').qemu(900).config.post(memory=8192)

# Add CPU
proxmox.nodes('pve1').qemu(900).config.post(cores=4)
```

### Clone VM

```python
proxmox.nodes('pve1').qemu(900).clone.post(
    newid=901,
    name='cloned-vm',
    full=1
)
```

### Delete VM

```python
proxmox.nodes('pve1').qemu(900).delete()
```

## Container Operations

### Create Container

```python
ct_config = {
    'vmid': 200,
    'ostype': 'debian',
    'hostname': 'my-container',
    'rootfs': 'local:8',
    'memory': 2048,
    'cores': 2,
    'net0': 'name=eth0,bridge=vmbr0,ip=dhcp'
}

proxmox.nodes('pve1').lxc.post(**ct_config)
```

### Start Container

```python
proxmox.nodes('pve1').lxc(200).status.start.post()
```

### Container Shell

```python
# Execute command
result = proxmox.nodes('pve1').lxc(200).status.post(
    command='apt update'
)
```

## Storage Operations

### List Storage

```python
storage = proxmox.storage.get()

for s in storage:
    print(f"{s['storage']}: {s['type']}")
```

### List Storage Content

```python
content = proxmox.storage('local').content.get()

for item in content:
    print(f"{item['volid']} ({item['size']})")
```

### Upload Template

```python
# Upload template
with open('/path/to/template.tar.gz', 'rb') as f:
    proxmox.storage('local').upload.post(
        filename='template.tar.gz',
        content=f.read()
    )
```

## User Management

### List Users

```python
users = proxmox.access.users.get()

for user in users:
    print(user['userid'])
```

### Create User

```python
proxmox.access.users.post(
    userid='newuser@example.com',
    password='password123'
)
```

### Create API Token

```python
proxmox.access.users('root@pam').tokens.post(
    tokenid='my-script-token'
)
```

## Access Control

### List ACLs

```python
acls = proxmox.access.acl.get()

for acl in acls:
    print(f"{acl['path']} - {acl['roleid']}")
```

### Add Permission

```python
proxmox.access.acl.post(
    path='/vms/100',
    userid='admin@example.com',
    roleid='PVEUser'
)
```

## Pool Management

### Create Pool

```python
proxmox.pool.post(
    poolid='web-servers',
    comment='Web server VMs'
)
```

### Add to Pool

```python
proxmox.pool('web-servers').put(vmid=100)
```

## Error Handling

```python
import proxmoxer
from proxmoxer.tools import ProxmoxHTTPErrors

try:
    proxmox.nodes('pve1').qemu(900).status.start.post()
except ProxmoxHTTPErrors as e:
    if e.status_code == 500:
        print("VM already running")
    else:
        print(f"Error: {e.message}")
```

## Advanced

### Custom Headers

```python
proxmox = proxmoxer.ProxmoxAPI(
    'https://pve.example.com:8006',
    user='root@pam',
    password='password',
    headers={'X-Custom-Header': 'value'}
)
```

### Timeout

```python
import requests

session = requests.Session()
session.timeout = 30

proxmox = proxmoxer.ProxmoxAPI(
    'https://pve.example.com:8006',
    user='root@pam',
    password='password',
    session=session
)
```

## Keywords

#python #api #automation #proxmoxer #scripting

## Related Articles

- [[API]]
- [[API-Authentication]]
- [[Scripts]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
