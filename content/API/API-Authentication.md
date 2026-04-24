# API Authentication Complete Guide

## Overview

Proxmox VE API supports multiple authentication methods. This guide covers all authentication options for secure API access.

## Authentication Methods

### 1. API Tokens (Recommended)

#### Create Token

```bash
# Create token for user
pveum user token add root@pam automation-token

# Create with expiration
pveum user token add root@pam limited-token --expire 2025-12-31

# Create with privilege separation
pveum user token add root@pam read-only-token --privsep 1
```

#### Use Token

```bash
# Header authentication
curl -H "Authorization: PVEAPIToken=root@pam=automation-token" \
  https://pve.example.com:8006/api2/json/cluster/status

# Query parameter (not recommended)
curl "https://pve.example.com:8006/api2/json/cluster/status?token=PVEAPIToken=root@pam=automation-token"
```

### 2. Basic Authentication

```bash
# Basic auth
curl -u "root@pam:password" \
  https://pve.example.com:8006/api2/json/cluster/status

# Username format
# root@pam - local user
# root@pamrealm - LDAP/AD user
```

### 3. Ticket Authentication

```bash
# Get ticket
curl -c cookies.txt -u "root@pam:password" \
  https://pve.example.com:8006/api2/json/access/ticket

# Use ticket
curl -b cookies.txt \
  https://pve.example.com:8006/api2/json/cluster/status
```

### 4. OAuth 2.0

```bash
# Get OAuth token
curl -X POST https://oauth.example.com/token \
  -d "grant_type=client_credentials" \
  -d "client_id=proxmox" \
  -d "client_secret=secret"

# Use token
curl -H "Authorization: Bearer <token>" \
  https://pve.example.com:8006/api2/json/cluster/status
```

## Token Management

### List Tokens

```bash
# List all tokens
pveum user token list root@pam

# Show token info
pveum user token info root@pam automation-token
```

### Regenerate Token

```bash
# Regenerate token (shows new secret)
pveum user token Regen root@pam automation-token
```

### Delete Token

```bash
# Delete token
pveum user token delete root@pam automation-token
```

## Security Best Practices

### Privilege Separation

```bash
# Create token with limited privileges
pveum user token add root@pam backup-token --privsep 1

# Grant specific access
pveum acl modify /vms/ --tokenid 'root@pam=backup-token' --role PVEUser
pveum acl modify /storage/local --tokenid 'root@pam=backup-token' --role PVEAudit
```

### Token Expiration

```bash
# Set expiration
pveum user token add root@pam temp-token --expire 2025-06-30
```

### Token Rotation

```bash
# Rotate quarterly
# 1. Create new token
pveum user token add root@pam q2-token
# 2. Update scripts
# 3. Delete old token
pveum user token delete root@pam q1-token
```

## API Client Libraries

### Python

```python
import proxmoxer

# Using token
proxmox = proxmoxer.ProxmoxAPI(
    'https://pve.example.com:8006',
    user='root@pam',
    token_password='automation-token',
    verify_ssl=False
)

# List VMs
vms = proxmox.cluster/resources.get()
```

### Perl

```perl
use Proxmox::API;

my $api = Proxmox::API->new(
    host => 'pve.example.com',
    user => 'root@pam',
    token => 'automation-token',
);
```

### Go

```go
package main

import "github.com/lcab/proxmox-go"

func main() {
    client := proxmox.NewClient("https://pve.example.com:8006", proxmox.TokenCreds)
    
    vms, _ := client.ClusterResources()
}
```

### JavaScript/Node.js

```javascript
const ProxmoxAPI = require('proxmox-api');

const proxmox = new ProxmoxAPI('https://pve.example.com:8006', {
    token: 'root@pam=automation-token'
});

const vms = await proxmox.cluster.resources.get();
```

## Error Responses

### 401 Unauthorized

```json
{
  "data": null,
  "errors": [{
    "code": 401,
    "message": "authentication failed"
  }]
}
```

### 403 Forbidden

```json
{
  "data": null,
  "errors": [{
    "code": 403,
    "message": "permission denied"
  }]
}
```

## Troubleshooting

### Invalid Token

```bash
# Verify token exists
pveum user token list root@pam

# Check token secret
pveum user token info root@pam automation-token
```

### Permission Denied

```bash
# Check ACLs
pveum acl list

# Check token permissions
pveum acl list / --tokenid 'root@pam=automation-token'
```

## Keywords

#api #authentication #token #oauth #basic-auth #security

## Related Articles

- [[API]]
- [[Scripts]]
- [[Access-Management]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
