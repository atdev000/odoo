# Odoo 19.0 Architecture Document

## Executive Summary

Odoo is a comprehensive open-source ERP (Enterprise Resource Planning) system built on Python with a modular addon architecture. Version 19.0 represents the latest stable release, featuring 600 business modules, a custom ORM, and a full-stack web framework.

## Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Language | Python 3.10-3.13 | Core application |
| Database | PostgreSQL 13+ | Data persistence |
| Web Framework | Werkzeug | WSGI/HTTP handling |
| Async Runtime | Gevent | Concurrency |
| ORM | Custom Odoo ORM | Database abstraction |
| Template Engine | QWeb + Jinja2 | View rendering |
| Frontend | OWL (JavaScript) | Reactive UI |
| CSS | LibSass/SCSS | Styling |

## Architecture Pattern

**Layered MVC with Plugin Architecture**

```
┌─────────────────────────────────────────────────────────────────┐
│                      PRESENTATION LAYER                          │
│  ┌───────────────┐  ┌───────────────┐  ┌─────────────────────┐  │
│  │  Web Client   │  │  QWeb Views   │  │   Static Assets     │  │
│  │   (OWL/JS)    │  │    (XML)      │  │  (CSS/JS/Images)    │  │
│  └───────────────┘  └───────────────┘  └─────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                       HTTP/CONTROLLER LAYER                      │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  Werkzeug WSGI → odoo/http.py → @route Controllers          ││
│  │  • Request/Response handling                                 ││
│  │  • Session management                                        ││
│  │  • Authentication (public/user/none)                        ││
│  └─────────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────┤
│                     BUSINESS LOGIC LAYER                         │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  600 Addon Modules                                          ││
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐           ││
│  │  │ Account │ │  Sale   │ │   HR    │ │  Stock  │  ...      ││
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘           ││
│  │                                                              ││
│  │  Models → Services → Wizards → Reports                      ││
│  └─────────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────┤
│                      DATA ACCESS LAYER                           │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  Odoo ORM (odoo/orm/)                                       ││
│  │  • Models & Fields                                          ││
│  │  • Domains & Environments                                   ││
│  │  • Registry & Caching                                       ││
│  │                    ↓                                         ││
│  │  psycopg2 → PostgreSQL                                      ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

## Component Architecture

### 1. Core Framework (`odoo/`)

```
odoo/
├── http.py          # WSGI application, request routing
├── orm/             # Object-Relational Mapper
│   ├── models.py    # Base model classes
│   ├── fields.py    # Field type system
│   └── registry.py  # Model registry
├── cli/             # Command-line interface
├── service/         # Background services
└── tools/           # Utility functions
```

### 2. Addon Module Architecture

Each addon is self-contained:

```
addon/
├── __manifest__.py  # Dependencies, metadata
├── models/          # Business logic (ORM)
├── views/           # UI definitions (XML)
├── controllers/     # HTTP endpoints
├── security/        # Access control
├── data/            # Configuration data
├── static/          # Frontend assets
└── tests/           # Unit tests
```

### 3. Request Flow

```
HTTP Request
    │
    ▼
┌─────────────────┐
│  Werkzeug WSGI  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   http.py       │
│  Application    │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐ ┌───────┐
│Static │ │Dynamic│
│ Files │ │Routes │
└───────┘ └───┬───┘
              │
         ┌────┴────┐
         ▼         ▼
    ┌────────┐ ┌────────┐
    │ No DB  │ │With DB │
    │ Routes │ │ Routes │
    └───┬────┘ └───┬────┘
        │          │
        ▼          ▼
    ┌────────┐ ┌────────┐
    │Dispatch│ │ir.http │
    └───┬────┘ │ _match │
        │      └───┬────┘
        │          │
        ▼          ▼
    ┌─────────────────┐
    │   Controller    │
    │   @route        │
    │   endpoint      │
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │   Response      │
    │ (JSON/HTML/PDF) │
    └─────────────────┘
```

## ORM Architecture

### Model Hierarchy

```
BaseModel (abstract)
    │
    ├── Model (persistent, stored in DB)
    │   └── Business models (res.partner, sale.order, etc.)
    │
    ├── TransientModel (temporary, auto-vacuumed)
    │   └── Wizard models (export dialogs, etc.)
    │
    └── AbstractModel (not stored, mixin only)
        └── Mixin classes (mail.thread, etc.)
```

### Field Types

| Category | Types |
|----------|-------|
| Simple | Char, Text, Html, Integer, Float, Boolean |
| Temporal | Date, Datetime |
| Binary | Binary, Image |
| Relational | Many2one, One2many, Many2many |
| Special | Selection, Reference, Monetary, Json |

### Environment

```python
# Environment = cursor + user + context
env = self.env

# Access models
partners = env['res.partner']

# Change user
admin_env = env(user=1)

# Change context
with_context = env.with_context(lang='es_ES')

# Sudo (bypass access rights)
sudo_env = env.sudo()
```

## Security Model

### Access Control Layers

```
┌─────────────────────────────────────────┐
│           Record Rules (ir.rule)         │
│     Row-level security with domains      │
├─────────────────────────────────────────┤
│        Model Access (ir.model.access)    │
│     CRUD permissions per model/group     │
├─────────────────────────────────────────┤
│              User Groups                 │
│     Role-based access groupings          │
├─────────────────────────────────────────┤
│           Authentication                 │
│     Session, OAuth, LDAP, 2FA            │
└─────────────────────────────────────────┘
```

### Security File Example

```csv
# ir.model.access.csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_my_model_user,my.model.user,model_my_model,base.group_user,1,1,1,0
access_my_model_manager,my.model.manager,model_my_model,sales_team.group_sale_manager,1,1,1,1
```

## Multi-tenancy

### Multi-Company

```
┌─────────────────────────────────────────┐
│              Database                    │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │Company A│  │Company B│  │Company C│ │
│  └─────────┘  └─────────┘  └─────────┘ │
│         All in same DB instance         │
└─────────────────────────────────────────┘
```

- **company_dependent** fields store per-company values
- **check_company=True** ensures record access within company
- Users can access multiple companies

### Multi-Database

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Database 1 │  │  Database 2 │  │  Database 3 │
│  (Client A) │  │  (Client B) │  │  (Client C) │
└─────────────┘  └─────────────┘  └─────────────┘
        │                │                │
        └────────────────┼────────────────┘
                         │
              ┌──────────────────┐
              │   Odoo Server    │
              │  (Shared Code)   │
              └──────────────────┘
```

## Module Dependencies

### Core Module Graph

```
base (Required)
  │
  ├── mail (Messaging)
  │   ├── crm
  │   ├── sale
  │   └── project
  │
  ├── product
  │   ├── sale
  │   ├── purchase
  │   └── stock
  │
  ├── account (Accounting)
  │   ├── sale (invoicing)
  │   └── purchase (bills)
  │
  └── website (CMS)
      ├── website_sale (E-commerce)
      └── website_blog
```

## Data Architecture

### Database Schema Patterns

```
┌──────────────────┐      ┌──────────────────┐
│   res_partner    │      │   sale_order     │
├──────────────────┤      ├──────────────────┤
│ id               │◄─────┤ partner_id (FK)  │
│ name             │      │ name             │
│ company_id (FK)  │      │ state            │
│ create_uid (FK)  │      │ amount_total     │
│ write_uid (FK)   │      │ company_id (FK)  │
│ create_date      │      └────────┬─────────┘
│ write_date       │               │
└──────────────────┘               │
                                   │ 1:N
                          ┌────────▼─────────┐
                          │ sale_order_line  │
                          ├──────────────────┤
                          │ order_id (FK)    │
                          │ product_id (FK)  │
                          │ qty              │
                          │ price_unit       │
                          └──────────────────┘
```

### Common Table Patterns

- **Audit columns**: `create_uid`, `write_uid`, `create_date`, `write_date`
- **Company scoping**: `company_id` foreign key
- **Soft delete**: `active` boolean field
- **Sequence**: `sequence` integer for ordering

## Caching Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Request Cache                            │
│  Per-request memoization via Environment                     │
├─────────────────────────────────────────────────────────────┤
│                     Registry Cache                           │
│  Model definitions, computed field caches                    │
├─────────────────────────────────────────────────────────────┤
│                     Signaling Cache                          │
│  Cross-worker invalidation via database signaling            │
└─────────────────────────────────────────────────────────────┘
```

## Deployment Architecture

### Single Server

```
┌─────────────────────────────────────┐
│            Single Server            │
│  ┌───────────────────────────────┐ │
│  │       Odoo (Gevent)           │ │
│  │  Workers: 0 (async greenlets) │ │
│  └─────────────┬─────────────────┘ │
│                │                    │
│  ┌─────────────▼─────────────────┐ │
│  │        PostgreSQL             │ │
│  └───────────────────────────────┘ │
└─────────────────────────────────────┘
```

### Production (Multi-Worker)

```
┌────────────────────────────────────────────────────────────┐
│                    Load Balancer                            │
│                  (nginx/HAProxy)                           │
└─────────────────────────┬──────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼───────┐ ┌───────▼───────┐ ┌───────▼───────┐
│   Worker 1    │ │   Worker 2    │ │   Worker N    │
│   (Prefork)   │ │   (Prefork)   │ │   (Prefork)   │
└───────┬───────┘ └───────┬───────┘ └───────┬───────┘
        │                 │                 │
        └─────────────────┼─────────────────┘
                          │
              ┌───────────▼───────────┐
              │      PostgreSQL       │
              │   (Primary/Replica)   │
              └───────────────────────┘
```

## Extension Points

### Inheritance Types

1. **Class Inheritance** (`_inherit`): Add/modify fields and methods
2. **Prototype Inheritance** (`_inherits`): Delegate to parent record
3. **Mixin Classes**: Add capabilities (mail.thread, etc.)

### View Inheritance

```xml
<record id="view_partner_form_inherit" model="ir.ui.view">
    <field name="model">res.partner</field>
    <field name="inherit_id" ref="base.view_partner_form"/>
    <field name="arch" type="xml">
        <field name="email" position="after">
            <field name="custom_field"/>
        </field>
    </field>
</record>
```

## Performance Characteristics

| Metric | Typical Value |
|--------|---------------|
| Models loaded | 1,259+ |
| Tables created | 1,000+ |
| Request latency | 50-200ms |
| Memory per worker | 200-500MB |
| Concurrent users | 10-50 per worker |

## Integration Capabilities

- **XML-RPC**: Legacy API for external integrations
- **JSON-RPC**: Modern API for web clients
- **REST**: Via custom controllers
- **Webhooks**: Via base_automation
- **EDI**: Electronic data interchange modules
- **OAuth**: Provider and consumer support
