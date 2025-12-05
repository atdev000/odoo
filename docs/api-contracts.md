# Odoo 19.0 API Contracts

## Executive Overview

The Odoo codebase contains an extensive HTTP API architecture with **211+ @route decorators** distributed across **168 controller modules** organized in **600 addon packages**.

## Statistics

| Metric | Count |
|--------|-------|
| Total Addon Packages | 600 |
| Controller Modules | 168 |
| API Routes/Endpoints | 211+ |
| JSONRPC Endpoints | ~95% |
| HTTP Endpoints | ~5% |

## Route Type Distribution

| Type | Purpose |
|------|---------|
| `jsonrpc` | JSON-RPC for AJAX/async operations |
| `http` | Standard HTTP for page rendering |
| `pdf` | PDF document generation |

## Authentication Patterns

### Public Endpoints (`auth='public'`)
- E-commerce storefronts (`/shop/*`)
- Payment processing (`/payment/*`)
- Public portal access
- WebSocket updates
- Guest checkout

### Authenticated Endpoints (`auth='user'`)
- Account management (`/my/*`)
- Backend interface (`/web/*`)
- Sales operations (`/sale/*`)
- Financial operations (`/account/*`)
- CRM operations (`/lead/*`)

## Major API Endpoint Categories

### E-Commerce (website_sale) - 57 Endpoints
- **Cart**: `/shop/cart`, `/shop/cart/add`, `/shop/cart/update`, `/shop/cart/clear`
- **Delivery**: `/shop/delivery_methods`, `/shop/set_delivery_method`, `/shop/get_delivery_rate`
- **Payment**: `/shop/payment/transaction/{order_id}`
- **Checkout**: `/shop/extra_info`, `/shop/payment`, `/shop/payment/validate`, `/shop/confirmation`

### Portal - User Dashboards
- `/my` - User dashboard
- `/my/account` - Account details
- `/my/addresses` - Manage addresses
- `/my/security` - Security settings
- `/my/counters` - Badge counters (JSONRPC)

### Mail/Messaging
- `/discuss/gif/search` - GIF search
- `/discuss/settings/mute` - Mute notifications
- `/mail/message/translate` - Message translation
- `/websocket/update_bus_presence` - User presence

### Web Backend
- `/web/action/load` - Load action definition
- `/web/action/run` - Execute server action
- `/web/model/get_definitions` - Get model definitions
- `/web/view/edit_custom` - Edit custom views

### Account/Financial
- `/product/catalog/get_sections` - Catalog sections
- `/product/catalog/create_section` - Create section

### Sales Configuration
- `/sale/product_configurator/get_values` - Get variant values
- `/sale/product_configurator/create_product` - Create variant
- `/sale/combo_configurator/get_data` - Combo product data

### Payment Processing
- `/payment/pay` - Payment form
- `/payment/transaction` - Create transaction
- `/payment/confirmation` - Payment confirmation

### CRM
- `/lead/case_mark_won` - Mark lead as won
- `/lead/case_mark_lost` - Mark lead as lost
- `/lead/convert` - Convert lead to opportunity

## Common Route Decorator Parameters

| Parameter | Purpose | Common Values |
|-----------|---------|---------------|
| `type` | Request/response type | jsonrpc, http, pdf |
| `auth` | Authentication level | public, user, none |
| `methods` | HTTP methods allowed | POST, GET |
| `website` | Website-specific context | True/False |
| `readonly` | Read-only operation | True/False |
| `cors` | CORS headers | "*" |

## URL Naming Conventions

```
/shop/       - E-commerce endpoints
/my/         - Portal/user endpoints
/discuss/    - Messaging endpoints
/web/        - Backend UI endpoints
/payment/    - Payment processing
/sale/       - Sales configuration
/lead/       - CRM endpoints
/mail/       - Email endpoints
/product/    - Product management
/website/    - Website management
```

## Data Flow Example: E-Commerce

```
GET /shop/                      (Browse products)
  |
JSONRPC /shop/cart/add          (Add to cart)
  |
GET /shop/cart                  (View cart)
  |
JSONRPC /shop/set_delivery_method (Set shipping)
  |
GET /shop/payment               (Payment page)
  |
JSONRPC /shop/payment/transaction (Create payment)
  |
GET /shop/payment/validate      (Process payment)
  |
GET /shop/confirmation          (Confirmation)
```

## Security Patterns

- **Token validation** for external access via `access_token` parameter
- **Context restriction** checking `request.website.has_ecommerce_access()`
- **Data access verification** via record existence and access checks
- **CORS support** limited to specific endpoints (`cors="*"`)
