# Security Best Practices for Access Management

## Overview

This guide covers essential security best practices for maintaining secure access control in your Proxmox VE environment.

## User Account Security

### Password Policies

```bash
# Configure password requirements in /etc/pve/user.cfg
minpwlen: 12
minpwqual: 3
```

| Quality Level | Requirements |
|---------------|---------------|
| 0 | No requirements |
| 1 | Minimum length only |
| 2 | Length + not username |
| 3 | Length + complexity |
| 4 | Maximum complexity |

### Password Rotation

```bash
# Force password change
pveum user passwd admin@example.com

# Set password expiration
pveum user update admin@example.com --expire 2025-06-30
```

## Authentication Hardening

### Two-Factor Authentication

**Required for:**
- All administrative accounts
- All remote access accounts
- Service accounts used for automation

```bash
# Verify 2FA status
pveum user 2fa list admin@example.com
```

### Account Lockout

```bash
# Configure account lockout
# In /etc/pve/user.cfg
maxfailed: 5
locktime: 30
```

| Setting | Description | Recommended |
|---------|-------------|-------------|
| maxfailed | Failed attempts before lockout | 3-5 |
| locktime | Lockout duration (minutes) | 15-30 |

## Access Control Principles

### Principle of Least Privilege

```bash
# Instead of granting broad access:
pveum acl modify /vms/ --user dev@example.com --role PVEAdmin

# Grant specific access:
pveum acl modify /vms/200 --user dev@example.com --role PVEUser
pveum acl modify /vms/200 --user dev@example.com --role PVEPowerAdmin
```

### Separation of Duties

Create distinct roles for different functions:

```bash
# VM Operator - can start/stop VMs
pveum role add VMOperator --privelist "VM.PowerMgmt"

# VM Admin - can configure VMs
pveum role add VMAdmin --privelist "VM.Config,VM.PowerMgmt"

# Auditor - read-only access
# Use built-in PVEAudit role
```

## Network Security

### Proxmox Firewall

```bash
# Enable datacenter firewall
pvesh create /nodes/localhost/firewall/options --enable 1

# Block admin access from outside
pvesh create /nodes/localhost/firewall/groups --group admin \
  --comment "Admin network only"

# Allow only from management network
pvesh create /nodes/localhost/rules/100 --action ACCEPT \
  --source 10.0.0.0/8 --proto tcp --dport 8006
```

### SSL/TLS Enforcement

```bash
# Force HTTPS
# In Datacenter → SSL Certificate → Enforce HTTPS

# Configure TLS options
# /etc/pve/pve-proxy.cfg
cipher: TLSv1.3
ciphers: ECDHE-RSA-AES256-GCM-SHA384
```

## API Security

### API Token Management

```bash
# Create limited-scope tokens
pveum user token add admin@example.com backup \
  --privsep 1 --expire 2025-12-31

# Grant minimal permissions
pveum acl modify /vms/ --tokenid \
  'admin@example.com=backup' --role PVEPowerAdmin

# Use token for specific tasks
pveum acl modify /storage/local \
  --tokenid 'admin@example.com=backup' --role PVEAudit
```

### API Rate Limiting

```bash
# Configure rate limiting
# In Datacenter → Options → API Rate Limit
# Default: 100 requests per minute
# Adjust based on needs
```

## Monitoring and Alerts

### Access Alerts

Configure alerts for:
- Failed login attempts (3+ in 15 minutes)
- Administrative actions
- Permission changes
- New user creation
- 2FA disable events

```bash
# Enable security alerts
# Datacenter → Logs → Alerts → Add
# Category: Security
# Events: login-failed, permission-change, user-create
```

### Regular Reviews

| Review | Frequency | Owner |
|--------|-----------|-------|
| User account audit | Monthly | Security team |
| Permission review | Quarterly | Team leads |
| Access certification | Semi-annual | Management |
| Configuration audit | Annual | Compliance |

## Incident Response

### Compromised Account Response

1. **Immediately disable account**
   ```bash
   pveum user update compromised@example.com --enable 0
   ```

2. **Revoke all tokens**
   ```bash
   pveum user token delete compromised@example.com --all
   ```

3. **Review access logs**
   ```bash
   pveam access log --user compromised@example.com --start 2025-01-01
   ```

4. **Remove permissions**
   ```bash
   pveum acl delete /vms/ --user compromised@example.com
   ```

5. **Investigate and remediate**

## Security Checklist

- [ ] Unique user IDs for each user
- [ ] 2FA enabled for all users
- [ ] Strong password policies enforced
- [ ] Least privilege applied
- [ ] Groups used for permission management
- [ ] Regular access reviews scheduled
- [ ] Audit logging enabled
- [ ] API tokens with expiration
- [ ] Service accounts reviewed quarterly
- [ ] Emergency contact procedures documented

## Regulatory Compliance

### Common Requirements

| Framework | Requirements |
|-----------|---------------|
| HIPAA | 2FA, audit logs, access reviews |
| PCI DSS | Unique IDs, logging, 2FA |
| SOX | Access certification, logging |
| GDPR | Access control, data minimization |

## Related Articles

- [[User Management]]
- [[Two-Factor Authentication]]
- [[Access Auditing]]
- [[Roles and Permissions]]