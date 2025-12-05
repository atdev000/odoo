# Odoo 19.0 Security Guide

## Authentication Mechanisms

### Password Authentication

- **Algorithm**: PBKDF2-SHA512
- **Minimum Rounds**: 600,000 (configurable)
- **Auto-upgrade**: Passwords upgraded on login

```python
# Configuration parameter
password.hashing.rounds = 600000
```

### Multi-Factor Authentication (2FA)

| Method | Module | Description |
|--------|--------|-------------|
| TOTP | `auth_totp` | Time-based one-time passwords |
| Passkey/WebAuthn | `auth_passkey` | FIDO2 hardware keys |
| Email OTP | `auth_totp_mail` | Email verification codes |

### OAuth/SSO

| Module | Providers |
|--------|-----------|
| `auth_oauth` | Google, Facebook, Odoo.com |
| `auth_ldap` | LDAP/Active Directory |

### API Keys

```python
# Generate API key for user
user.api_key_ids.create({
    'name': 'My Integration',
    'scope': 'rpc',  # or specific scope
    'expiration_date': '2025-12-31',
})
```

- Scoped access control
- Configurable expiration
- PBKDF2-SHA512 hashed storage

## Access Control Layers

### Layer 1: User Groups

```xml
<!-- Define a group -->
<record id="group_my_manager" model="res.groups">
    <field name="name">My Module Manager</field>
    <field name="category_id" ref="base.module_category_hidden"/>
    <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
</record>
```

**Predefined Groups:**
- `base.group_system` - Administrator
- `base.group_user` - Internal User (Employee)
- `base.group_portal` - Portal User
- `base.group_public` - Public User

### Layer 2: Model Access (ir.model.access)

```csv
# security/ir.model.access.csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_my_model_user,my.model.user,model_my_model,base.group_user,1,0,0,0
access_my_model_manager,my.model.manager,model_my_model,group_my_manager,1,1,1,1
```

| Permission | Description |
|------------|-------------|
| `perm_read` | Can read records |
| `perm_write` | Can update records |
| `perm_create` | Can create records |
| `perm_unlink` | Can delete records |

### Layer 3: Record Rules (ir.rule)

```xml
<!-- User can only see own records -->
<record id="rule_my_model_user" model="ir.rule">
    <field name="name">My Model: User own records</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="domain_force">[('user_id', '=', user.id)]</field>
    <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    <field name="perm_read" eval="True"/>
    <field name="perm_write" eval="True"/>
    <field name="perm_create" eval="True"/>
    <field name="perm_unlink" eval="True"/>
</record>

<!-- Multi-company rule -->
<record id="rule_my_model_company" model="ir.rule">
    <field name="name">My Model: Company</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    <field name="global" eval="True"/>
</record>
```

**Domain Variables:**
- `user` - Current user record
- `company_id` - Current company
- `company_ids` - Allowed companies

## HTTP Route Security

### Auth Modes

```python
from odoo import http

class MyController(http.Controller):

    @http.route('/public', type='http', auth='public')
    def public_page(self):
        """Accessible without login"""
        pass

    @http.route('/user', type='http', auth='user')
    def user_page(self):
        """Requires authenticated user"""
        pass

    @http.route('/none', type='http', auth='none')
    def no_auth_page(self):
        """No authentication (use carefully)"""
        pass
```

| Auth Mode | Description |
|-----------|-------------|
| `public` | Public access, creates public user session |
| `user` | Requires logged-in user |
| `none` | No authentication at all |

### CSRF Protection

- Enabled by default for POST/PUT/DELETE
- Token lifetime: 1 year
- Uses HMAC with database secret

```python
# Disable CSRF for specific route (use carefully)
@http.route('/webhook', type='json', auth='none', csrf=False)
def webhook(self):
    pass
```

## Security Best Practices

### Model Security

```python
class MyModel(models.Model):
    _name = 'my.model'

    # Restrict field access
    secret_field = fields.Char(groups='base.group_system')

    # Company-aware fields
    company_id = fields.Many2one('res.company', required=True,
                                  default=lambda self: self.env.company)

    @api.model
    def create(self, vals):
        # Validate input before creation
        if 'dangerous_field' in vals:
            raise AccessError("Cannot set dangerous_field directly")
        return super().create(vals)
```

### SQL Injection Prevention

```python
# WRONG - SQL Injection vulnerable
self.env.cr.execute("SELECT * FROM res_partner WHERE name = '%s'" % name)

# CORRECT - Use parameterized queries
self.env.cr.execute("SELECT * FROM res_partner WHERE name = %s", (name,))

# BEST - Use ORM methods
self.env['res.partner'].search([('name', '=', name)])
```

### XSS Prevention

```python
# Fields that render HTML safely
description = fields.Html(sanitize=True)  # Default: sanitized

# Disable sanitization only when necessary
trusted_html = fields.Html(sanitize=False)  # Use with caution
```

### Sudo Usage

```python
# Use sudo() sparingly - bypasses access rights
record.sudo().write({'field': 'value'})

# Prefer with_user() for impersonation
record.with_user(specific_user).write({'field': 'value'})

# Always validate before sudo operations
if not self.env.user.has_group('my_module.group_manager'):
    raise AccessError("Not authorized")
record.sudo().sensitive_operation()
```

## Session Management

| Setting | Default | Description |
|---------|---------|-------------|
| Session Lifetime | 7 days | Maximum session duration |
| Rotation Interval | 3 hours | Token rotation frequency |
| Grace Period | 120 seconds | Post-rotation validity |

## Security Configuration

### Important Parameters

```
# ir.config_parameter
password.hashing.rounds = 600000
database.secret = <auto-generated>
auth.session.max_inactivity_seconds = 7776000
```

### Audit Logging

Security events automatically logged:
- Login success/failure with IP
- Password changes
- 2FA verification attempts
- API key creation/removal
- Access denied events

## Checklist

### For New Modules

- [ ] Define `ir.model.access.csv` for all models
- [ ] Create record rules for multi-company/user isolation
- [ ] Set appropriate `groups` on sensitive fields
- [ ] Use `auth='user'` for authenticated routes
- [ ] Sanitize HTML fields
- [ ] Use parameterized SQL queries
- [ ] Validate user input
- [ ] Document security model in README
