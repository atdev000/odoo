# Odoo 19.0 Integration Guide

## API Protocols

| Protocol | Endpoint | Auth | Use Case |
|----------|----------|------|----------|
| JSON-RPC 2.0 | `/jsonrpc` | Session | Legacy clients |
| JSON/2 | `/json/2/<model>/<method>` | Bearer | Modern API |
| XML-RPC | `/xmlrpc/2/<service>` | Basic | Legacy systems |
| HTTP REST | Custom routes | Various | Web/mobile apps |

## Authentication Methods

### Session Authentication (Web)
```python
@http.route('/my/endpoint', type='http', auth='user')
def my_endpoint(self):
    # Requires logged-in user session
    user = request.env.user
```

### Bearer Token Authentication (API)
```python
@http.route('/api/v1/data', type='json2', auth='bearer')
def api_data(self, **kwargs):
    # Requires Authorization: Bearer <token>
    return {'data': self.env['my.model'].search_read([])}
```

### API Key Management
```python
# Create API key for user
api_key = self.env['res.users.apikeys'].create({
    'name': 'My Integration',
    'user_id': user.id,
    'scope': 'rpc',
})
key_value = api_key._generate()
```

### Authentication Modes
| Mode | Description |
|------|-------------|
| `user` | Requires authenticated session |
| `public` | Optional authentication |
| `none` | No authentication |
| `bearer` | API key/token required |

## JSON-RPC 2.0 API

### Request Format
```json
{
    "jsonrpc": "2.0",
    "method": "call",
    "params": {
        "model": "res.partner",
        "method": "search_read",
        "args": [[["is_company", "=", true]]],
        "kwargs": {"fields": ["name", "email"], "limit": 10}
    },
    "id": 1
}
```

### Response Format
```json
{
    "jsonrpc": "2.0",
    "result": [
        {"id": 1, "name": "Company A", "email": "a@example.com"}
    ],
    "id": 1
}
```

### Python Client Example
```python
import requests

url = "https://odoo.example.com/jsonrpc"
payload = {
    "jsonrpc": "2.0",
    "method": "call",
    "params": {
        "service": "object",
        "method": "execute_kw",
        "args": [
            "database",
            uid,
            "password",
            "res.partner",
            "search_read",
            [[["is_company", "=", True]]],
            {"fields": ["name"], "limit": 5}
        ]
    },
    "id": 1
}
response = requests.post(url, json=payload)
```

## Custom HTTP Controllers

### Basic Controller
```python
from odoo import http
from odoo.http import request

class MyAPI(http.Controller):

    @http.route('/api/partners', type='json', auth='user', methods=['POST'])
    def get_partners(self, domain=None, limit=100):
        partners = request.env['res.partner'].search_read(
            domain or [],
            fields=['name', 'email', 'phone'],
            limit=limit
        )
        return {'status': 'success', 'data': partners}

    @http.route('/api/partner/<int:partner_id>', type='http', auth='user')
    def get_partner(self, partner_id):
        partner = request.env['res.partner'].browse(partner_id)
        if not partner.exists():
            return request.not_found()
        return request.make_json_response({
            'id': partner.id,
            'name': partner.name
        })
```

### CORS Support
```python
@http.route('/api/public', type='json', auth='none', cors='*')
def public_endpoint(self):
    return {'status': 'ok'}
```

### CSRF Configuration
```python
# Disable CSRF for webhooks
@http.route('/webhook', type='json', auth='none', csrf=False)
def webhook_handler(self, **payload):
    return {'received': True}
```

## WebSocket / Real-Time (Bus)

### Server-Side Publishing
```python
# Send notification to specific user
self.env['bus.bus']._sendone(
    (self._cr.dbname, 'res.users', user_id),
    'notification',
    {'message': 'Hello!'}
)

# Broadcast to channel
self.env['bus.bus']._sendone(
    (self._cr.dbname, 'my_channel'),
    'update',
    {'data': 'content'}
)
```

### Model-Based Broadcasting
```python
class MyModel(models.Model):
    _name = 'my.model'
    _inherit = ['bus.listener.mixin']

    def notify_update(self):
        self._bus_send('record_updated', {
            'id': self.id,
            'name': self.name
        })
```

### JavaScript Client (OWL)
```javascript
import { useService } from "@web/core/utils/hooks";

setup() {
    this.busService = useService("bus_service");
    this.busService.subscribe("my_channel", (payload) => {
        console.log("Received:", payload);
    });
}
```

## OAuth Integration

### Provider Configuration
```xml
<record id="oauth_google" model="auth.oauth.provider">
    <field name="name">Google</field>
    <field name="client_id">your-client-id</field>
    <field name="auth_endpoint">https://accounts.google.com/o/oauth2/v2/auth</field>
    <field name="validation_endpoint">https://www.googleapis.com/oauth2/v3/tokeninfo</field>
    <field name="scope">openid profile email</field>
</record>
```

### OAuth Flow
1. User clicks OAuth login button
2. Redirect to provider's auth endpoint
3. Provider redirects back to `/auth_oauth/signin`
4. Odoo validates token and creates/logs in user

## Webhook Patterns

### Receiving Webhooks
```python
class WebhookController(http.Controller):

    @http.route('/webhook/payment', type='json', auth='none', csrf=False)
    def payment_webhook(self, **payload):
        # Validate webhook signature
        if not self._verify_signature(payload):
            return {'error': 'Invalid signature'}

        # Process payment notification
        self.env['payment.transaction'].sudo()._handle_webhook(payload)
        return {'status': 'ok'}

    def _verify_signature(self, payload):
        # Implement signature verification
        return True
```

### Sending Webhooks (Server Actions)
```xml
<record id="action_webhook_notify" model="ir.actions.server">
    <field name="name">Send Webhook</field>
    <field name="state">webhook</field>
    <field name="webhook_url">https://external.api/webhook</field>
    <field name="webhook_field_ids" eval="[(4, ref('field_my_model__name'))]"/>
</record>
```

## External API Calls

### Using Requests
```python
import requests

class MyModel(models.Model):
    _name = 'my.model'

    def call_external_api(self):
        response = requests.post(
            'https://api.example.com/data',
            json={'record_id': self.id},
            headers={'Authorization': f'Bearer {self.api_key}'},
            timeout=30
        )
        response.raise_for_status()
        return response.json()
```

## Key Files

| File | Purpose |
|------|---------|
| `odoo/http.py` | HTTP/WSGI layer |
| `addons/rpc/controllers/` | RPC endpoints |
| `addons/bus/` | WebSocket/real-time |
| `addons/auth_oauth/` | OAuth integration |
| `odoo/addons/base/models/res_users.py` | API keys |

## Security Best Practices

1. **Use `auth='user'` or `auth='bearer'`** for sensitive endpoints
2. **Validate webhook signatures** before processing
3. **Rate limit API endpoints** via reverse proxy
4. **Use HTTPS** for all external communication
5. **Scope API keys** appropriately
6. **Log API access** for auditing
7. **Sanitize input data** from external sources
