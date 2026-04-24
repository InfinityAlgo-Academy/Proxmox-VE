---
title: Network Services
tags:
  - network
  - services
  - dns
  - dhcp
  - ldap
---

# Network Services

## Overview

Configure essential network services for Proxmox.

## DHCP Server

### Install

```bash
apt install isc-dhcp-server -y
```

### Configure

```bash
# /etc/dhcp/dhcpd.conf
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.50 192.168.1.100;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    default-lease-time 600;
    max-lease-time 7200;
}
```

### Start

```bash
systemctl start isc-dhcp-server
```

## DNS Server

- See [[DNS-Config]]

## LDAP/AD Integration

- See [[LDAP-AD-Integration]]

## NTP Server

```bash
# Install chrony
apt install chrony -y

# Configure
nano /etc/chrony/chrony.conf

# Allow clients
allow 192.168.1.0/24
```

## Services for VMs

| Service | Port | Description |
|---------|------|-------------|
| DHCP | 67 | IP assignment |
| DNS | 53 | Name resolution |
| NTP | 123 | Time sync |
| LDAP | 389 | Directory |

## See Also

- [[index|Back to Proxmox VE]]
- [[Network-Config]]