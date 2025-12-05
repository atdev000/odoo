# Odoo 19.0 Data Models

## Overview

| Metric | Value |
|--------|-------|
| Total Unique Models | 1,259+ |
| Total Addons | 600 |
| Core Infrastructure Models | 100+ |
| Business Models | 350+ |

## ORM Structure

**Location**: `odoo/orm/`

### Field Types

| Category | Fields |
|----------|--------|
| **Textual** | `Char`, `Text`, `Html` |
| **Numeric** | `Integer`, `Float`, `Monetary` |
| **Temporal** | `Date`, `Datetime` |
| **Relational** | `Many2one`, `One2many`, `Many2many` |
| **Selection** | `Selection`, `Reference`, `Many2oneReference` |
| **Binary** | `Binary`, `Image` |
| **Other** | `Boolean`, `Json`, `Id`, `Properties` |

### Base Model Classes

- **Model** - Persistent database models
- **TransientModel** - Temporary in-memory models
- **AbstractModel** - Non-persistent base classes

## Core Infrastructure Models (ir.*, res.*)

### IR Models (System Infrastructure)

**Model Management**:
- `ir.model` - Model metadata registry
- `ir.model.fields` - Field definitions
- `ir.model.access` - Access control rules
- `ir.model.data` - External ID mappings

**Module Management**:
- `ir.module.module` - Installed modules
- `ir.module.category` - Module categories

**UI Components**:
- `ir.ui.view` - View definitions (forms, trees, kanban)
- `ir.ui.menu` - Menu structure
- `ir.actions.act_window` - Window actions

**System Utilities**:
- `ir.cron` - Scheduled jobs
- `ir.attachment` - File attachments
- `ir.config_parameter` - System configuration
- `ir.sequence` - Auto-incrementing sequences
- `ir.rule` - Row-level security

### RES Models (Core Resources)

**Users & Groups**:
- `res.users` - User accounts
- `res.groups` - User groups/roles

**Organization**:
- `res.company` - Legal entities
- `res.partner` - Customers, vendors, contacts
- `res.country` - Countries
- `res.currency` - Currencies

## Business Domain Models

### Accounting (`account`) - 38 Models

| Model | Purpose |
|-------|---------|
| `account.move` | Journal entries, invoices, bills |
| `account.move.line` | Line items |
| `account.account` | GL accounts |
| `account.journal` | Journal types |
| `account.payment` | Payment records |
| `account.tax` | Tax definitions |
| `account.bank.statement` | Bank statements |
| `account.reconcile.model` | Auto-reconciliation |

### Sales (`sale`) - 3 Models

| Model | Purpose |
|-------|---------|
| `sale.order` | Sales orders/quotations |
| `sale.order.line` | Order line items |

### Purchase (`purchase`) - 3 Models

| Model | Purpose |
|-------|---------|
| `purchase.order` | Purchase orders/RFQs |
| `purchase.order.line` | Order line items |

### CRM (`crm`) - 7 Models

| Model | Purpose |
|-------|---------|
| `crm.lead` | Leads and opportunities |
| `crm.stage` | Pipeline stages |
| `crm.team` | Sales teams |

### Human Resources (`hr`) - 11 Models

| Model | Purpose |
|-------|---------|
| `hr.employee` | Employee records |
| `hr.department` | Departments |
| `hr.job` | Job positions |

### Inventory (`stock`) - 22 Models

| Model | Purpose |
|-------|---------|
| `stock.picking` | Stock transfers |
| `stock.move` | Stock movements |
| `stock.quant` | Stock quantities |
| `stock.location` | Locations |
| `stock.warehouse` | Warehouses |

### Project (`project`) - 11 Models

| Model | Purpose |
|-------|---------|
| `project.project` | Projects |
| `project.task` | Tasks |
| `project.milestone` | Milestones |

### Manufacturing (`mrp`) - 14 Models

| Model | Purpose |
|-------|---------|
| `mrp.production` | Manufacturing orders |
| `mrp.bom` | Bill of materials |
| `mrp.workorder` | Work orders |
| `mrp.workcenter` | Work centers |

### Mail/Messaging (`mail`) - 50+ Models

| Model | Purpose |
|-------|---------|
| `mail.message` | Messages/emails |
| `mail.thread` | Threading mixin |
| `mail.activity` | Activities |
| `discuss.channel` | Chat channels |

### Website (`website`) - 30+ Models

| Model | Purpose |
|-------|---------|
| `website` | Website instances |
| `website.page` | Web pages |
| `website.menu` | Navigation menus |
| `website.visitor` | Visitor tracking |

## Inheritance Patterns

### Mixin-Based (`_inherit`)

Common mixins:
- `mail.thread` - Messaging/threading
- `mail.activity.mixin` - Activity management
- `portal.mixin` - Customer portal access
- `utm.mixin` - Campaign tracking
- `avatar.mixin` - Profile images

### Delegation (`_inherits`)

```python
# hr.employee inherits hr.version
_inherits = {'hr.version': 'version_id'}
```

## Common Field Patterns

```python
# Tracking changes
tracking=True

# Computed fields
compute='_compute_name'
store=True
inverse='_inverse_name'

# Relationships
check_company=True
ondelete='cascade'

# Indexing
index=True
index='trigram'
```

## Key Relationships

### Central Hub: res.partner
```
res.partner connects to:
- res.users (user accounts)
- res.company (organizations)
- account.move (invoices)
- sale.order (sales)
- purchase.order (purchases)
- crm.lead (opportunities)
- stock.picking (deliveries)
```

### Financial Hub: account.move
```
account.move connects to:
- account.journal
- account.account
- account.payment
- sale.order
- purchase.order
```

## Model Naming Conventions

| Prefix | Domain |
|--------|--------|
| `account.*` | Accounting/Financial |
| `res.*` | Core resources |
| `ir.*` | Internal/System |
| `stock.*` | Inventory |
| `sale.*` | Sales |
| `purchase.*` | Purchasing |
| `crm.*` | CRM |
| `hr.*` | Human resources |
| `project.*` | Projects |
| `mrp.*` | Manufacturing |
| `mail.*` | Messaging |
| `website.*` | Website/CMS |
