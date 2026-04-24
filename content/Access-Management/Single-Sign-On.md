# Single Sign-On (SSO) with Proxmox VE

## Overview

Single Sign-On allows users to authenticate once and access multiple services. Proxmox VE supports SSO through SAML 2.0 and OAuth 2.0 protocols.

## SSO Providers

### Supported Providers

- Okta
- Azure AD
- Google Workspace
- OneLogin
- Keycloak
- Auth0

## SAML 2.0 Configuration

### Service Provider (SP) Metadata

1. Navigate to **Datacenter → Authentication**
2. Download SP metadata:
   ```
   https://pve.example.com:8006/api2/extjs/pve-ext:sp-metadata.xml
   ```

The metadata provides:
- Entity ID: `pve.example.com`
- ACS URL: `https://pve.example.com:8006/api2/extjs/pve-ext:acs`
- SLO URL: `https://pve.example.com:8006/api2/extjs/pve-ext:slo`

### IdP Configuration

```bash
# Add SAML realm
pve realms add saml "okta" \
  --entity-id "https://pve.example.com:8006" \
  --idp-metadata "/tmp/okta-metadata.xml" \
  --autologin 1
```

### SAML User Attributes

| SAML Attribute | Proxmox Field |
|----------------|---------------|
| NameID | User ID |
| email | Email |
| givenName | Real Name |
| memberOf | Groups |

### SAML Group Mapping

```bash
# Map SAML groups to Proxmox roles
pve realms update okta --group-mapping "Proxmox-Admins=PVEAdmin,Proxmox-Users=PVEUser"
```

## OAuth 2.0 / OpenID Connect

### Configuring OAuth

```bash
# Add OAuth provider
pve realms add oauth2 "google" \
  --client-id "<client-id>" \
  --client-secret "<client-secret>" \
  --auth-uri "https://accounts.google.com/o/oauth2/v2/auth" \
  --token-uri "https://oauth2.googleapis.com/token"
```

### Azure AD OAuth Setup

1. Register app in Azure Portal
2. Configure redirect URIs:
   ```
   https://pve.example.com:8006/api2/extjs/pve-ext:oauth_callback
   ```
3. Request API permissions:
   - `User.Read`
   - `GroupMember.Read.All`

## SSO Login Flow

### User Experience

1. User visits Proxmox login page
2. Clicks "Sign in with SSO"
3. Redirected to IdP
4. Authenticates at IdP (if not already)
5. Redirected back to Proxmox
6. Access granted automatically

### Web UI SSO

```
https://pve.example.com:8006/
  → Redirect to IdP
  → Login at IdP
  → Redirect back with token
  → Auto-login user
```

### API SSO

```bash
# Get token from IdP
curl -L -c cookies.txt \
  "https://idp.example.com/authorize?\
client_id=proxmox&\
redirect_uri=https://pve.example.com:8006/api2/extjs/pve-ext:oauth_callback&\
response_type=code&\
scope=openid+profile+email"

# Exchange for API token
curl -L -b cookies.txt \
  -X POST "https://idp.example.com/token" \
  -d "grant_type=authorization_code" \
  -d "code=<auth-code>" \
  -d "redirect_uri=https://pve.example.com:8006/api2/extjs/pve-ext:oauth_callback"
```

## SSO Security

### Certificate Management

```bash
# Upload IdP certificate
pve realms update okta --idp-cert "/tmp/idp-cert.pem"

# Verify certificate
pve realms test okta --verify-cert
```

### Session Management

```bash
# SSO session timeout (default: 8 hours)
pve realms update okta --session-timeout 28800

# Force re-authentication
pve realms update okta --force-local-login 1
```

## Troubleshooting SSO

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Invalid signature | Certificate mismatch | Update IdP metadata |
| Audience mismatch | Wrong entity ID | Check SP entity ID |
| Attribute not found | Missing claim | Update IdP config |
| Group mapping failed | Invalid group | Verify group names |

### Debugging

```bash
# Enable SSO debugging
pve realms update okta --debug 1

# View SSO logs
journalctl -u pveproxy -n 50 | grep saml
```

## Related Articles

- [[LDAP AD Integration]]
- [[User Management]]
- [[Roles and Permissions]]
- [[Two-Factor Authentication]]