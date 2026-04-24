# Groups and Group Management in Proxmox VE

## Overview

Groups in Proxmox VE provide a way to organize users and manage permissions collectively. Instead of assigning permissions to individual users, you assign them to groups, simplifying administration.

## Creating Groups

### Via Web UI

1. Navigate to **Datacenter → Groups**
2. Click **Add**
3. Enter group name (e.g., developers, admins, contractors)
4. Add optional comment

### Via CLI

```bash
# Create a group
pveum group add developers --comment "Development team"

# Create admin group
pveum group add admins --comment "Administrator team"

# Create contractor group
pveum group add contractors --comment "External contractors"
```

## Managing Group Members

```bash
# Add user to group
pveum group adduser developers john@example.com

# Add multiple users
pveum group adduser developers jane@example.com
pveum group adduser developers bob@example.com

# Remove user from group
pveum group deluser developers john@example.com

# List group members
pveum group listuser developers
```

## Deleting Groups

```bash
# Delete group (removes group only, not users)
pveum group delete developers

# Delete and remove from all ACLs
pveum group delete developers --purge 1
```

## Group-Based Permissions

### Example: Development Team Access

```bash
# Create development group
pveum group add developers

# Add developers to group
pveum group adduser developers dev1@example.com
pveum group adduser developers dev2@example.com
pveum group adduser developers dev3@example.com

# Grant VM Power User access to development VMs
pveum acl modify /vms/200 --group developers --role PVEUser
pveum acl modify /vms/201 --group developers --role PVEUser
pveum group adduser developers dev2@example.com

# Grant storage access
pveum acl modify /storage/local --group developers --role PVEAudit
```

### Example: Operations Team Access

```bash
# Create operations group
pveum group add operations

# Add ops team members
pveum group adduser operations ops1@example.com
pveum group adduser operations ops2@example.com

# Grant full VM management
pveum acl modify /vms/ --group operations --role PVEPowerAdmin

# Grant storage management
pveum group adduser operations ops3@example.com
pveum acl modify /storage/ --group operations --role PVEAdmin
```

### Example: Contractor Restrictions

```bash
# Create contractor group
pveum group add contractors

# Add contractor
pveum group adduser contractors contractor@example.com

# Grant only read access to specific VM
pveum acl modify /vms/300 --group contractors --role PVEAudit

# Set expiration on contractor account
pveum user update contractor@example.com --expire 2025-03-31
```

## Nested Groups

Proxmox VE doesn't support nested groups directly, but you can simulate this with role inheritance:

```bash
# Create sub-groups
pveum group add frontend-devs
pveum group add backend-devs

# Add users to sub-groups
pveum group adduser frontend-devs john@example.com
pveum group adduser backend-devs jane@example.com

# Main group gets access to all sub-group resources
pveum acl modify /vms/100 --group frontend-devs --role PVEUser
pveum acl modify /vms/100 --group backend-devs --role PVEUser
```

## Group Auditing

```bash
# List all groups
pveum group list

# List members of a group
pveum group listuser developers

# List permissions for a group
pveum acl list / --group developers
```

## Best Practices

1. **Use descriptive names** - Name groups by function (e.g., "frontend-devs", "db-admins")
2. **Regular member reviews** - Audit group membership quarterly
3. **Separate production and development** - Different groups for different environments
4. **Document group purpose** - Use comments to explain group intent
5. **Use expiration** - Set account expiry for temporary team members

## Group Management Commands

| Task | Command |
|------|---------|
| Create group | `pveum group add <name>` |
| Delete group | `pveum group delete <name>` |
| Add user | `pveum group adduser <group> <user>` |
| Remove user | `pveum group deluser <group> <user>` |
| List members | `pveum group listuser <group>` |
| List groups | `pveum group list` |

## Related Articles

- [[Roles and Permissions]]
- [[User Management]]
- [[ACL Best Practices]]
- [[Access Auditing]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
