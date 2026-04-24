# Access Management Complete Guide

## User Management

### Create Users
```bash
# Via CLI
pveum user add admin@example.com --password secret

# Via Web UI
# Datacenter → Users → Add
```

### User Roles

| Role | Description |
|------|-------------|
| PVEAudit | Read-only access |
| PVEUser | Standard user |
| PVEPowerAdmin | Power user |
| PVEAdmin | Full administrator |

### Assign Permissions
```bash
# VM permissions
pveum acl modify /vms/100 --user admin@example.com --roles PVEUser

# Storage permissions
pveum acl modify /storage/local --user admin@example.com --roles PVEUser
```

### API Tokens

```bash
# Create token
pveum user token add admin@example.com mytoken

# List tokens
pveum user token list admin@example.com

# Use token
curl -H "Authorization: PVEAPIToken=admin@example.com=mytoken" \
  https://pve:8006/api2/json/
```

### Two-Factor Authentication

```bash
# Enable 2FA via Web UI
# User → 2FA → Add → TOTP
```

## Links

- [[Security]]