# JavaScript/Node.js API Client

## Overview

Use Node.js to interact with Proxmox VE API for web-based automation and integrations.

## Setup

### Install Library

```bash
npm install proxmox-api
```

### Using Yarn

```bash
yarn add proxmox-api
```

## Basic Usage

### Connect to API

```javascript
const ProxmoxAPI = require('proxmox-api');

const proxmox = new ProxmoxAPI({
    host: 'https://pve.example.com:8006',
    user: 'root@pam',
    password: 'password'
});

// Or using token
const proxmox = new ProxmoxAPI({
    host: 'https://pve.example.com:8006',
    user: 'root@pam',
    token: 'your-token'
});
```

### Async/Await

```javascript
const ProxmoxAPI = require('proxmox-api');

async function main() {
    const proxmox = new ProxmoxAPI({
        host: 'https://pve.example.com:8006',
        user: 'root@pam',
        token: 'your-token'
    });
    
    const status = await proxmox.cluster.status();
    console.log(status);
}

main().catch(console.error);
```

## Cluster Operations

### Get Cluster Status

```javascript
const status = await proxmox.cluster.status();
status.forEach(node => {
    console.log(`${node.node}: ${node.status}`);
});
```

### Get Resources

```javascript
const resources = await proxmox.cluster.resources();
const vms = resources.filter(r => r.type === 'qemu');
const containers = resources.filter(r => r.type === 'lxc');
```

## VM Operations

### List VMs

```javascript
const vms = await proxmox.cluster.resources();
vms.filter(r => r.type === 'qemu').forEach(vm => {
    console.log(`VM ${vm.vmid}: ${vm.name} (${vm.status})`);
});
```

### Create VM

```javascript
const newVM = await proxmox.nodes('pve1').qemu.create({
    vmid: 900,
    name: 'my-new-vm',
    memory: 4096,
    cores: 2,
    ostype: 'l26',
    scsi0: 'local:32',
    net0: 'virtio,bridge=vmbr0'
});
```

### Start VM

```javascript
await proxmox.nodes('pve1').qemu(900).status.start();
```

### Stop VM

```javascript
await proxmox.nodes('pve1').qemu(900).status.stop();
```

### Get VM Config

```javascript
const config = await proxmox.nodes('pve1').qemu(900).config.get();
console.log(config);
```

### Update VM

```javascript
await proxmox.nodes('pve1').qemu(900).config.update({
    memory: 8192,
    cores: 4
});
```

### Clone VM

```javascript
await proxmox.nodes('pve1').qemu(900).clone({
    newid: 901,
    name: 'cloned-vm',
    full: 1
});
```

### Delete VM

```javascript
await proxmox.nodes('pve1').qemu(900).delete();
```

## Container Operations

### Create Container

```javascript
await proxmox.nodes('pve1').lxc.create({
    vmid: 200,
    ostype: 'debian',
    hostname: 'my-container',
    rootfs: 'local:8',
    memory: 2048,
    cores: 2,
    net0: 'name=eth0,bridge=vmbr0,ip=dhcp'
});
```

### Start Container

```javascript
await proxmox.nodes('pve1').lxc(200).status.start();
```

### Execute Command

```javascript
const result = await proxmox.nodes('pve1').lxc(200).status.post({
    command: 'apt update'
});
```

## Storage Operations

### List Storage

```javascript
const storage = await proxmox.storage.get();
storage.forEach(s => {
    console.log(`${s.storage}: ${s.type}`);
});
```

### List Storage Content

```javascript
const content = await proxmox.storage('local').content.get();
content.forEach(item => {
    console.log(`${item.volid}: ${item.size}`);
});
```

## User Operations

### List Users

```javascript
const users = await proxmox.access.users.get();
users.forEach(user => {
    console.log(user.userid);
});
```

### Create User

```javascript
await proxmox.access.users.create({
    userid: 'newuser@example.com',
    password: 'password123'
});
```

### Create Token

```javascript
const token = await proxmox.access.users('root@pam').tokens.create({
    tokenid: 'my-script-token'
});
console.log(token);
```

## Error Handling

```javascript
try {
    await proxmox.nodes('pve1').qemu(900).status.start();
} catch (error) {
    console.error('Error:', error.message);
    console.error('Code:', error.statusCode);
}
```

## Express.js Integration

```javascript
const express = require('express');
const app = express();
const ProxmoxAPI = require('proxmox-api');

const proxmox = new ProxmoxAPI({
    host: process.env.PVE_HOST,
    user: 'root@pam',
    token: process.env.PVE_TOKEN
});

app.get('/vms', async (req, res) => {
    try {
        const vms = await proxmox.cluster.resources();
        res.json(vms.filter(r => r.type === 'qemu'));
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

app.post('/vms/:vmid/start', async (req, res) => {
    try {
        const vmid = req.params.vmid;
        await proxmox.nodes('pve1').qemu(vmid).status.start();
        res.json({ success: true });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

app.listen(3000);
```

## Keywords

#javascript #nodejs #api #automation #express

## Related Articles

- [[API]]
- [[API-Authentication]]
- [[API-Python-Client]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
