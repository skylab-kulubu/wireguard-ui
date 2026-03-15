# WireGuard UI Email Modification Plan

## Overview
This plan outlines enhancements to the WireGuard UI email automation system to make it more configurable, personalized, and automated.

## Implementation Status: ✅ COMPLETED

### Completed Features

#### Phase 1: Configuration Enhancements ✅
- ✅ **Email Subject Configurable**: `--email-subject` flag and `EMAIL_SUBJECT` env variable
- ✅ **Email Content Configurable**: `--email-content` flag and `EMAIL_CONTENT` env variable  
- ✅ **Email Template File Support**: `--email-template-file` flag to load templates from external files

#### Phase 2: Personalization ✅
- ✅ **Client Data Placeholders** - Supported placeholders:
  - `{{.ClientName}}` - Client's name
  - `{{.ClientEmail}}` - Client's email
  - `{{.CreatedAt}}` - Client creation date
  - `{{.UpdatedAt}}` - Client update date
  - `{{.PublicKey}}` - Client's public key
  - `{{.Endpoint}}` - Server endpoint
  - `{{.AllowedIPs}}` - Allowed IP ranges
  - `{{.ServerPublicKey}}` - Server public key
  - `{{.DNS}}` - DNS servers

#### Phase 3: Automation ✅
- ✅ **Auto-email on Client Creation**: `--auto-email-on-create` flag and `AUTO_EMAIL_ON_CREATE` env variable
- ✅ **Auto-email on Client Update**: `--auto-email-on-client-update` flag and `AUTO_EMAIL_ON_CLIENT_UPDATE` env variable

#### Phase 4: Template System ✅
- ✅ **Sample HTML Template**: Created `examples/email-template.html` with professional styling and all placeholder support

---

## Usage Guide

### Command Line Flags

```bash
# Email subject and content
./wireguard-ui --email-subject "Your VPN Configuration" --email-content "<p>Hello {{.ClientName}}, here is your config...</p>"

# Or use environment variables
export EMAIL_SUBJECT="Your VPN Configuration"
export EMAIL_CONTENT="<p>Hello {{.ClientName}}, here is your config...</p>"

# Load from template file
./wireguard-ui --email-template-file /path/to/template.html

# Auto-email on client creation/update
./wireguard-ui --auto-email-on-create --auto-email-on-client-update

# Or via environment variables
export AUTO_EMAIL_ON_CREATE=true
export AUTO_EMAIL_ON_CLIENT_UPDATE=true
```

### Available Placeholders

| Placeholder | Description |
|-------------|-------------|
| `{{.ClientName}}` | Client's name |
| `{{.ClientEmail}}` | Client's email address |
| `{{.CreatedAt}}` | Account creation timestamp |
| `{{.UpdatedAt}}` | Last update timestamp |
| `{{.PublicKey}}` | Client's public key |
| `{{.Endpoint}}` | VPN server endpoint |
| `{{.AllowedIPs}}` | Allowed IP ranges |
| `{{.ServerPublicKey}}` | Server's public key |
| `{{.DNS}}` | DNS servers |

---

## Backward Compatibility

- ✅ All changes maintain backward compatibility
- ✅ Default values match existing behavior
- ✅ Existing API endpoints unchanged
- ✅ No breaking changes to email sending interface

---

## Files Modified

1. `main.go` - Added email configuration flags and auto-email triggers
2. `handler/routes.go` - Added PersonalizeEmail function and updated handlers
3. `examples/email-template.html` - Sample email template
