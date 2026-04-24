# User Management in Proxmox VE

## Overview

Proxmox VE provides a comprehensive user management system built on top of Linux PAM (Pluggable Authentication Modules). Users can be created, modified, and deleted either through the web interface or CLI.

## Creating Users

### Via Web UI

1. Navigate to **Datacenter → Users**
2. Click **Add**
3. Fill in the required fields:
   - **User name**: Full email address (e.g., admin@example.com)
   - **Real name**: Display name
   - **Password**: Secure password
   - **Groups**: Select relevant groups
   - **Enabled**: Check to enable
   - **Expiry**: Set expiration date (optional)

### Via CLI

```bash
# Create basic user
pveum user add admin@example.com --password secret123

# Create user with real name
pveum user add john.doe@example.com --password "SecurePass123" --realname "John Doe"

# Create user with email
pveum user add admin@example.com --password secret123 --email "admin@example.com"

# Set user expiration
pveum user add temp@example.com --password temp123 --expire 2025-12-31
```

## Modifying Users

```bash
# Change password
pveum user passwd admin@example.com

# Enable/disable user
pveum user update admin@example.com --enable 0
pveum user update admin@example.com --enable 1

# Set user comment
pveum user update admin@example.com --comment "Primary administrator"

# Change real name
pveum user update admin@example.com --realname "John Smith"
```

## Deleting Users

```bash
# Delete user (with confirmation prompt)
pveum user delete admin@example.com

# Delete user without prompt
pveum user delete admin@example.com --purge 0
```

## User Properties

| Property | Description | CLI Flag |
|----------|-------------|----------|
| User ID | Unique identifier (email format) | `user add <userid>` |
| Password | Authentication credential | `--password` |
| Real Name | Human-readable name | `--realname` |
| Email | Contact email | `--email` |
| Enabled | Account active status | `--enable` |
| Expiry | Account expiration date | `--expire` |
| Comment | Additional notes | `--comment` |

## User Authentication

### Password Requirements

Configure password policies in `/etc/pve/user.cfg`:

```
minpwlen: 8
minpwqual: 2
```

- `minpwlen`: Minimum password length
- `minpwqual`: Minimum password quality (0-4)

### Checking User Status

```bash
# List all users
pveum user list

# List specific user details
pveum user list admin@example.com

# Show user info
pveum user metrics admin@example.com
```

## Best Practices

1. **Use email-format user IDs** - Follows best practices for LDAP integration
2. **Enable 2FA** - Add two-factor authentication for all users
3. **Set expiration** - Use temporary accounts for contractors
4. **Strong passwords** - Enforce minimum length and complexity
5. **Regular audits** - Review user list quarterly

## Related Articles

- [[Roles and Permissions]]
- [[Two-Factor Authentication]]
- [[API Tokens]]
- [[LDAP Integration]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
