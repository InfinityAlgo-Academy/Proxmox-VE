---
title: API Complete Guide
---

# API Complete Guide

## REST API v2

### Base URL
```
https://<node>:8006/api2/json/
```

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /cluster/status | Cluster status |
| GET | /cluster/resources | All resources |
| GET | /nodes | Node list |
| GET | /nodes/{node}/status | Node info |
| GET | /storage | Storage list |
| GET | /access/users | User list |
| POST | /nodes/{node}/qemu | Create VM |
| POST | /nodes/{node}/lxc | Create CT |

### Authentication

#### API Token
```bash
# Create token
pveum user token add root@pam mytoken

# Use token
curl -H "Authorization: PVEAPIToken=root@pam=mytoken" \
  https://pve:8006/api2/json/cluster/status
```

#### Basic Auth
```bash
curl -u "root@pam:password" \
  https://pve:8006/api2/json/cluster/status
```

### Examples

#### List All VMs
```python
import requests

url = "https://pve:8006/api2/json/cluster/resources"
headers = {"Authorization": "PVEAPIToken=root@pam=mytoken"}
response = requests.get(url, headers=headers)
vms = [r for r in response.json()["data"] if r["type"] == "qemu"]
```

#### Start VM
```python
import requests

url = "https://pve:8006/api2/json/nodes/pve/qemu/100/status/start"
headers = {"Authorization": "PVEAPIToken=root@pam=mytoken"}
requests.post(url, headers=headers)
```

#### Create Container
```python
import requests

url = "https://pve:8006/api2/json/nodes/pve/lxc"
headers = {"Authorization": "PVEAPIToken=root@pam=mytoken"}
data = {
    "vmid": 200,
    "ostype": "debian",
    "rootfs": "local:8",
    "memory": 2048,
    "cores": 2,
    "net0": "name=eth0,bridge=vmbr0,ip=dhcp"
}
requests.post(url, headers=headers, json=data)
```

### PVE Shell Commands

```bash
# Using pvesh
pvesh get /cluster/status
pvesh get /nodes
pvesh create /nodes/pve/qemu --vmid 100
```

### Error Handling

```json
{
  "data": null,
  "errors": [
    {
      "code": 400,
      "message": "VM already exists"
    }
  ]
}
```

## Links

- [[Proxmox-VE]] - Main documentation
- [[Scripts]] - Automation scripts
---

[[index|Back to Proxmox VE]]
