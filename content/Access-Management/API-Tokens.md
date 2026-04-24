---
title: API Tokens in Proxmox VE
---

# API Tokens in Proxmox VE

## Overview

API tokens provide a way to authenticate with the Proxmox VE API without using passwords. They are ideal for automation, scripts, and third-party integrations.

## Token Structure

API tokens follow this format:
```
user_id=token_id
```

Full token value (used in API calls):
```
PVEAPIToken=user_id=token_id
```

## Creating API Tokens

### Via Web UI

1. Navigate to **Datacenter → Users**
2. Select user → **Tokens** tab
3. Click **Add**
4. Configure:
   - **Token ID**: Descriptive name (e.g., backup-script, monitoring)
   - **Privilege Separation**: Enable for limited access
   - **Expiration**: Optional expiry date

### Via CLI

```bash
# Create basic token
pveum user token add admin@example.com backup-script

# Create token with expiration
pveum user token add admin@example.com monitoring --expire 2025-12-31

# Create token with privilege separation
pveum user token add admin@example.com automation --privsep 1
```

## Using API Tokens

### With curl

```bash
# Using token in header
curl -H "Authorization: PVEAPIToken=admin@example.com=backup-script" \
  https://pve.example.com:8006/api2/json/nodes

# Using query parameter
curl "https://pve.example.com:8006/api2/json/nodes?token=PVEAPIToken=admin@example.com=backup-script"
```

### With Proxmox Perl/Python Libraries

```perl
# Perl
use Proxmox::API;
my $api = Proxmox::API->new(
    host => 'pve.example.com',
    token => 'admin@example.com=backup-script',
);
```

```python
# Python
import proxmoxer
proxmox = proxmoxer.ProxmoxAPI(
    'pve.example.com',
    user='admin@example.com',
    token_password='backup-script',
    verify_ssl=False
)
```

### With Terraform

```hcl
provider "proxmox" {
  pm_api_url = "https://pve.example.com:8006/api2/json"
  pm_api_token_id = "admin@example.com/terraform"
  pm_api_token_secret = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

## Token Management

```bash
# List tokens for user
pveum user token list admin@example.com

# Show token info (without secret)
pveum user token info admin@example.com backup-script

# Regenerate token (show new secret)
pveum user token Regen backup-script

# Delete token
pveum user token delete admin@example.com backup-script

# Update token expiration
pveum user token update admin@example.com monitoring --expire 2026-01-01
```

## Token Privileges

### Privilege Separation

When enabled, the token uses only the privileges assigned to the token, not the user's full privileges.

```bash
# Create token with privilege separation
pveum user token add admin@example.com limited-token --privsep 1

# Token can only do what it's specifically granted
pveum acl modify /vms/100 --tokenid 'admin@example.com=limited-token' --role PVEUser
```

### Default Privileges

Without privilege separation, tokens inherit all user privileges.

## Security Best Practices

1. **Use descriptive token IDs** -Easy identification and revocation
2. **Set expiration** - Limit token lifetime
3. **Enable privilege separation** - Limit token capabilities
4. **Store securely** - Use secrets management
5. **Rotate regularly** - Regenerate tokens periodically

## Token Security

```bash
# Check token last used
pveum user token info admin@example.com backup-script

# View token usage (audit)
# Navigate to: Datacenter → Users → [User] → Tokens → [Token]
```

## Environment Variables

```bash
# Set environment variables
export PMPROXY_TOKEN="PVEAPIToken=admin@example.com=backup-script"
export PMPROXY_HOST="https://pve.example.com:8006"

# Use with proxmox VE tools
pvesh nodes/$PMPROXY_HOST
```

## Troubleshooting

### Invalid Token Error

```
401 Unauthorized: invalid token
```

Solutions:
- Verify token ID format: `user_id=token_id`
- Check token not expired
- Confirm token not revoked

### Permission Denied

```
403 Forbidden: permission denied
```

Solutions:
- Check user has required permissions
- Verify privilege separation settings
- Grant additional ACLs if needed

## Related Articles

- [[API and API Access]]
- [[User Management]]
- [[Roles and Permissions]]
- [[Automation Scripts]]
---

[[index|Back to Proxmox VE]]
