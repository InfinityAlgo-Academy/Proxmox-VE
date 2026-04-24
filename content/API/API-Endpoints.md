# API Endpoints Complete Reference

## Overview

Complete reference for all Proxmox VE REST API v2 endpoints with parameters and responses.

## Base URL

```
https://<host>:8006/api2/json/
```

## Cluster Endpoints

### Cluster Status

```bash
GET /cluster/status
```

Response:
```json
{
  "data": [
    {
      "type": "node",
      "node": "pve1",
      "status": "online",
      "uptime": 86400,
      "cpu": 0.05,
      "mem": 8589934592,
      "maxmem": 17179869184,
      "disk": 107374182400,
      "maxdisk": 214748364800
    }
  ]
}
```

### Cluster Resources

```bash
GET /cluster/resources
```

Parameters:
- `type`: node, vm, storage

### Cluster HA Status

```bash
GET /cluster/ha/status
```

### Cluster Backup

```bash
GET /cluster/backup
POST /cluster/backup
```

## Node Endpoints

### Node Status

```bash
GET /nodes/{node}/status
```

### Node subscriptions

```bash
GET /nodes/{node}/subscription
```

### Node Scan

```bash
GET /nodes/{node}/scan
```

### Node DNS

```bash
GET /nodes/{node}/dns
PUT /nodes/{node}/dns
```

### Node APT

```bash
GET /nodes/{node}/apt/update
GET /nodes/{node}/apt/changelog
POST /nodes/{node}/apt/update
```

### Node Disk

```bash
GET /nodes/{node}/disk
POST /nodes/{node}/disk
```

## VM Endpoints (QEMU)

### List VMs

```bash
GET /nodes/{node}/qemu
```

### Create VM

```bash
POST /nodes/{node}/qemu
```

Parameters:
- `vmid`: VM ID
- `name`: VM name
- `memory`: Memory in MB
- `cores`: Number of cores
- `cpu`: CPU type
- `ostype`: OS type
- `scsi0`: Storage:size
- `net0`: Network config

### VM Status

```bash
GET /nodes/{node}/qemu/{vmid}/status
POST /nodes/{node}/qemu/{vmid}/status/start
POST /nodes/{node}/qemu/{vmid}/status/stop
POST /nodes/{node}/qemu/{vmid}/status/shutdown
POST /nodes/{node}/qemu/{vmid}/status/reset
POST /nodes/{node}/qemu/{vmid}/status/suspend
```

### VM Config

```bash
GET /nodes/{node}/qemu/{vmid}/config
PUT /nodes/{node}/qemu/{vmid}/config
```

### VM Monitor

```bash
GET /nodes/{node}/qemu/{vmid}/monitor
POST /nodes/{node}/qemu/{vmid}/monitor
```

### VM VNC WebSocket

```bash
GET /nodes/{node}/qemu/{vmid}/vncwebsocket
```

### VM Clone

```bash
POST /nodes/{node}/qemu/{vmid}/clone
```

Parameters:
- `newid`: Target VM ID
- `name`: New name
- `full`: Full clone (1/0)

### VM Template

```bash
POST /nodes/{node}/qemu/{vmid}/template
```

### VM Migrate

```bash
POST /nodes/{node}/qemu/{vmid}/migrate
```

Parameters:
- `target`: Target node
- `online`: 1 for live migration
- `storage`: Target storage

## Container Endpoints (LXC)

### List Containers

```bash
GET /nodes/{node}/lxc
```

### Create Container

```bash
POST /nodes/{node}/lxc
```

Parameters:
- `vmid`: Container ID
- `ostype`: debian, ubuntu, centos, alpine
- `hostname`: Hostname
- `rootfs`: Storage:size
- `memory`: Memory in MB
- `cores`: CPU cores
- `net0`: Network config

### Container Status

```bash
GET /nodes/{node}/lxc/{vmid}/status
POST /nodes/{node}/lxc/{vmid}/status/start
POST /nodes/{node}/lxc/{vmid}/status/stop
POST /nodes/{node}/lxc/{vmid}/status/restart
```

### Container Config

```bash
GET /nodes/{node}/lxc/{vmid}/config
PUT /nodes/{node}/lxc/{vmid}/config
```

## Storage Endpoints

### List Storage

```bash
GET /storage
POST /storage
DELETE /storage/{storage-id}
```

### Storage Content

```bash
GET /storage/{storage-id}/content
```

### Storage Status

```bash
GET /storage/{storage-id}/status
```

### Storage-Remove

```bash
POST /storage/{storage-id}/prune
```

## Access/User Endpoints

### List Users

```bash
GET /access/users
POST /access/users
DELETE /access/users/{userid}
```

### User Config

```bash
GET /access/users/{userid}
PUT /access/users/{userid}
```

### List Groups

```bash
GET /access/groups
POST /access/groups
```

### List ACLs

```bash
GET /access/acl
POST /access/acl
DELETE /access/acl
```

### Tickets

```bash
POST /access/ticket
```

## Pool Endpoints

### List Pools

```bash
GET /pool
POST /pool
```

### Pool Members

```bash
GET /pool/{poolid}
POST /pool/{poolid}
DELETE /pool/{poolid}/{vmid}
```

## API Tokens

### List Tokens

```bash
GET /access/users/{userid}/tokens
POST /access/users/{userid}/tokens
DELETE /access/users/{userid}/tokens/{tokenid}
```

## Version Endpoints

### API Version

```bash
GET /version
```

## Network Endpoints

### List Network

```bash
GET /nodes/{node}/network
POST /nodes/{node}/network
DELETE /nodes/{node}/network/{iface}
```

## Firewall Endpoints

### Firewall Rules

```bash
GET /nodes/{node}/firewall/rules
POST /nodes/{node}/firewall/rules
```

### Firewall Aliases

```bash
GET /nodes/{node}/firewall/aliases
POST /nodes/{node}/firewall/aliases
```

## Keywords

#api #rest #endpoints #reference #qemu #lxc #storage #cluster

## Related Articles

- [[API]]
- [[API-Authentication]]
- [[Scripts]]
---

[[index|Back to Proxmox VE]]
