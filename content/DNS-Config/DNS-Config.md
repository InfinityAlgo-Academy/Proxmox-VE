---
title: DNS Configuration
tags:
  - dns
  - network
  - dns- config
  - resolution
---

# DNS Configuration

## Overview

Configure DNS for Proxmox VE and all VMs/containers.

## Proxmox DNS

### /etc/resolv.conf

```bash
# Set nameserver
echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "nameserver 8.8.4.4" >> /etc/resolv.conf

# Search domain
echo "search local.domain" >> /etc/resolv.conf
```

## DNS for VM

```bash
# Set DNS for VM
qm set 100 --dns 8.8.8.8

# Set DNS search
qm set 100 --dns-search local.domain
```

## DNS for Container

```bash
# Set DNS for container
pct set 200 --dns 8.8.8.8
pct set 200 --dns-search local.domain
```

## Local DNS Server

### Install Bind9

```bash
apt install bind9 -y

# Configure
nano /etc/bind/named.conf.local
```

### Forward Zone

```zone "proxmox.local" {
    type master;
    file "/etc/bind/db.proxmox";
};
```

### Reverse Zone

```zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192";
};
```

## DNS Commands

| Command | Description |
|---------|-------------|
| `nslookup` | Query DNS |
| `dig` | Query DNS |
| `host` | DNS lookup |
| `getent hosts` | Host lookup |

## Troubleshooting DNS

- No resolution: Check /etc/resolv.conf
- Slow resolution: Check DNS server
- Wrong domain: Check search domain

## See Also

- [[index|Back to Proxmox VE]]
- [[Network]]
- [[DNS]]