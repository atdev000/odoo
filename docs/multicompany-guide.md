# Odoo 19.0 Multi-Company Guide

## Making Models Company-Aware

### Basic Implementation
```python
class MyModel(models.Model):
    _name = 'my.model'
    _check_company_auto = True  # Auto-validate company consistency

    company_id = fields.Many2one(
        'res.company',
        string='Company',
        required=True,
        index=True,
        default=lambda self: self.env.company
    )

    # Relational fields with company checking
    partner_id = fields.Many2one('res.partner', check_company=True)
    warehouse_id = fields.Many2one('stock.warehouse', check_company=True)
```

## Company-Dependent Fields

### Definition
```python
class ResPartner(models.Model):
    _inherit = 'res.partner'

    # Value varies per company (stored as JSONB)
    property_account_payable = fields.Many2one(
        'account.account',
        company_dependent=True,
        check_company=True,
        domain="[('account_type', '=', 'liability_payable')]"
    )

    property_payment_term = fields.Many2one(
        'account.payment.term',
        company_dependent=True
    )

    credit_limit = fields.Float(company_dependent=True)
```

### Storage Pattern
```
Database column stores JSONB:
{
    "1": account_id_for_company_1,
    "2": account_id_for_company_2
}
```

### Fallback Values
```python
# Set default via ir.default
self.env['ir.default'].set(
    'res.partner',
    'property_payment_term',
    payment_term.id,
    company_id=company.id
)
```

## Record Rules (ir.rule)

### Basic Multi-Company Rule
```xml
<record id="my_model_company_rule" model="ir.rule">
    <field name="name">My Model: Multi-Company</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="domain_force">[('company_id', 'in', company_ids)]</field>
</record>
```

### Allow Shared Records (company_id = False)
```xml
<record id="my_model_rule" model="ir.rule">
    <field name="name">My Model: Multi-Company + Shared</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="domain_force">
        [('company_id', 'in', company_ids + [False])]
    </field>
</record>
```

### Hierarchical Access (Parent Company)
```xml
<record id="my_model_parent_rule" model="ir.rule">
    <field name="name">My Model: Include Parent Company</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="domain_force">
        ['|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]
    </field>
</record>
```

### Rule Variables
| Variable | Description |
|----------|-------------|
| `company_ids` | User's allowed company IDs |
| `company_id` | Current active company ID |
| `user` | Current user record |

## Company Switching

### In Code
```python
# Switch to specific company context
other_company_record = record.with_company(other_company)

# Access company-dependent field in different company
with_company_2 = partner.with_company(company_2)
account = with_company_2.property_account_payable

# Get current company
current = self.env.company

# Get all user's companies
all_companies = self.env.companies
```

### Context Management
```python
# Context variable
allowed_company_ids = self.env.context.get('allowed_company_ids', [])

# First element is active company
active_company = self.env.company  # = companies[0]
```

## Company Hierarchy (Branches)

### Structure
```python
class ResCompany(models.Model):
    _name = 'res.company'

    parent_id = fields.Many2one('res.company', 'Parent Company')
    child_ids = fields.One2many('res.company', 'parent_id', 'Branches')
    parent_path = fields.Char(index=True)  # Tree traversal
    root_id = fields.Many2one(compute='_compute_parent_ids')
```

### Access Child Companies
```python
# Get accessible branches
branches = company._accessible_branches()

# Check if all branches selected
all_selected = company._all_branches_selected()
```

### Root Company Delegation
```python
# These fields are delegated to root company
# Changing on child affects root
def _get_company_root_delegated_field_names(self):
    return ['currency_id']
```

## Inter-Company Transactions

### Stock Inter-Company Location
```python
# Automatic inter-company transit location
location = self.env.ref('stock.stock_location_inter_company')
```

### Cross-Company Access
```python
# Access records from another company (with sudo)
other_records = self.env['my.model'].sudo().with_company(other_company).search([])
```

## Company Validation

### Automatic Validation
```python
class MyModel(models.Model):
    _check_company_auto = True  # Enable auto-check

    # On create/write, validates:
    # - All check_company=True fields point to compatible records
```

### Manual Validation
```python
def _check_company(self, fnames=None):
    """Manually trigger company consistency check"""
    super()._check_company(fnames)
```

### Custom Domain
```python
def _check_company_domain(self, companies):
    """Override for custom company filtering"""
    return [('company_id', 'in', companies.ids + [False])]
```

## User Company Access

### User Configuration
```python
class ResUsers(models.Model):
    _inherit = 'res.users'

    company_id = fields.Many2one('res.company')  # Default company
    company_ids = fields.Many2many('res.company')  # Allowed companies
```

### Check User Access
```python
# User can access company?
can_access = company in self.env.user.company_ids

# User's current company
current = self.env.company
```

## Key Files

| File | Purpose |
|------|---------|
| `odoo/addons/base/models/res_company.py` | Company model |
| `odoo/addons/base/models/ir_rule.py` | Record rules |
| `odoo/orm/fields.py` | company_dependent implementation |
| `odoo/orm/models.py` | _check_company logic |
| `odoo/addons/base/security/base_security.xml` | Base rules |

## Best Practices

1. **Always add `company_id`** to business models
2. **Set `_check_company_auto = True`** for validation
3. **Use `check_company=True`** on relational fields
4. **Create ir.rule** for multi-company filtering
5. **Use `company_dependent=True`** for per-company settings
6. **Include `+ [False]`** in rules for shared records
7. **Use `with_company()`** for context switching
8. **Test with multiple companies** during development

## Common Patterns

### Model with Company
```python
class SaleOrder(models.Model):
    _name = 'sale.order'
    _check_company_auto = True

    company_id = fields.Many2one(
        'res.company', required=True, index=True,
        default=lambda self: self.env.company
    )
    warehouse_id = fields.Many2one('stock.warehouse', check_company=True)
```

### Security Rule
```xml
<record id="sale_order_comp_rule" model="ir.rule">
    <field name="name">Sale Order: Multi-Company</field>
    <field name="model_id" ref="sale.model_sale_order"/>
    <field name="domain_force">[('company_id', 'in', company_ids)]</field>
</record>
```
