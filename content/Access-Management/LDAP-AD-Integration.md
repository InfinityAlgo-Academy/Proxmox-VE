---
title: LDAP and Active Directory Integration
---

# LDAP and Active Directory Integration

## Overview

Proxmox VE can integrate with LDAP directories for centralized user authentication. This simplifies management by using existing enterprise identity credentials.

## Supported Directories

- OpenLDAP
- Microsoft Active Directory
- Azure Active Directory
- Okta
- Google Workspace

## Configuring LDAP Realm

### Via Web UI

1. Navigate to **Datacenter → Authentication**
2. Click **Add → LDAP**
3. Configure:
   - **Realm**: LDAP
   - **Base DN**: `dc=example,dc=com`
   - **Server**: `ldap.example.com`
   - **Port**: 389 (or 636 for SSL)
   - **SSL/TLS**: Enable for production
   - **User Filter**: LDAP search filter

### Via CLI

```bash
# Add LDAP realm
pve realms add ldap --realm ldap.example.com \
  --base-dn "dc=example,dc=com" \
  --server1 ldap.example.com

# Add LDAP with SSL
pve realms add ldap --realm ldap.example.com \
  --base-dn "dc=example,dc=com" \
  --server1 ldap.example.com \
  --port 636 \
  --ssl verify
```

## Active Directory Configuration

### Basic AD Setup

```bash
# Add Active Directory realm
pve realms add active-directory --realm corp.example.com \
  --base-dn "dc=corp,dc=example,dc=com" \
  --server1 ad.corp.example.com \
  --domain corp.example.com

# Add with multiple servers
pve realms add active-directory --realm corp.example.com \
  --base-dn "dc=corp,dc=example,dc=com" \
  --server1 ad1.corp.example.com \
  --server2 ad2.corp.example.com \
  --domain corp.example.com
```

### AD Group Mapping

Sync AD groups to Proxmox:

```bash
# Sync users
pveum ad sync corp.example.com

# Sync with group filter
pveum ad sync corp.example.com --group "Domain Admins"
```

## LDAP/AD User Sync

### Manual Sync

```bash
# Sync all users
pveum ldap sync ldap.example.com

# Sync specific group
pveum ldap sync ldap.example.com --group "Proxmox-Users"

# Sync with role mapping
pveum ldap sync ldap.example.com --role PVEUser
```

### Auto-Sync Settings

```bash
# Enable automatic sync
pve realms update ldap --sync-config 1

# Set sync interval
pve realms update ldap --sync-interval 15
```

## LDAP Attributes

### Default User Attributes

| LDAP Attribute | Proxmox Field |
|---------------|---------------|
| uid | User ID |
| cn | Real Name |
| mail | Email |
| memberOf | Groups |

### Custom Attribute Mapping

```bash
# Configure custom attributes
pve realms update ldap --user-name-attr sAMAccountName \
  --real-name-attr displayName \
  --email-attr mail \
  --group-attr memberOf
```

## LDAP Authentication

### Bind DN Authentication

```bash
# Use bind DN for authentication
pve realms update ldap --bind-dn "cn=admin,dc=example,dc=com" \
  --bind-pass "password"
```

### Kerberos Authentication

```bash
# Enable Kerberos
pve realms update ldap --kerberos 1

# Configure keytab
pve realms update ldap --keytab /etc/krb5.keytab
```

## Troubleshooting LDAP

### Connection Test

```bash
# Test LDAP connection
pveum ldap test ldap.example.com

# Debug mode
pveum ldap test ldap.example.com --verbose
```

### Common Issues

| Error | Solution |
|-------|----------|
| Cannot connect | Check server/port/firewall |
| Authentication failed | Verify bind DN/password |
| Users not found | Check base DN and filter |
| Groups not syncing | Verify group attribute |

## Enterprise Integration

### Azure AD

```bash
# Add Azure AD
pve realms add azure --realm azure.example.com \
  --client-id "<application-id>" \
  --tenant-id "<tenant-id>" \
  --domain "example.onmicrosoft.com"
```

### Okta

Okta integration requires SAML or OAuth configuration.

## Security Considerations

1. **Use SSL/TLS** - Always encrypt LDAP traffic
2. **Service account** - Use dedicated bind account
3. **Least privilege** - Limit AD service account permissions
4. **Group filtering** - Only sync necessary groups

## Related Articles

- [[User Management]]
- [[Roles and Permissions]]
- [[Groups and Groups Management]]
- [[Single Sign-On]]
---

[[index|Back to Proxmox VE]]
