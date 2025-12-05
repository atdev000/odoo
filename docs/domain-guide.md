# Odoo 19.0 Domain Language Guide

## Domain Syntax

### Basic Structure
```python
# Single condition
[('field', 'operator', value)]

# Multiple conditions (implicit AND)
[('field1', '=', value1), ('field2', '!=', value2)]

# Explicit AND
['&', ('field1', '=', value1), ('field2', '=', value2)]

# OR
['|', ('field1', '=', value1), ('field2', '=', value2)]

# NOT
['!', ('field', '=', value)]
```

## Operators Reference

### Comparison Operators
| Operator | Description | Example |
|----------|-------------|---------|
| `=` | Equals | `('state', '=', 'draft')` |
| `!=` | Not equals | `('state', '!=', 'cancelled')` |
| `>` | Greater than | `('amount', '>', 100)` |
| `>=` | Greater or equal | `('date', '>=', '2025-01-01')` |
| `<` | Less than | `('quantity', '<', 10)` |
| `<=` | Less or equal | `('priority', '<=', 2)` |

### Collection Operators
| Operator | Description | Example |
|----------|-------------|---------|
| `in` | In list | `('state', 'in', ['draft', 'sent'])` |
| `not in` | Not in list | `('type', 'not in', ['internal'])` |

### String Operators
| Operator | Description | Example |
|----------|-------------|---------|
| `like` | Case-sensitive pattern | `('name', 'like', 'John')` |
| `ilike` | Case-insensitive pattern | `('email', 'ilike', '@gmail')` |
| `=like` | Exact pattern (no auto-wildcards) | `('code', '=like', 'SO-%')` |
| `=ilike` | Case-insensitive exact | `('ref', '=ilike', 'INV/%')` |
| `not like` | Negation | `('name', 'not like', 'test')` |
| `not ilike` | Case-insensitive negation | `('email', 'not ilike', 'spam')` |

### Relational Operators
| Operator | Description | Example |
|----------|-------------|---------|
| `any` | Match on related records | `('order_ids', 'any', [('state', '=', 'sale')])` |
| `not any` | No related match | `('invoice_ids', 'not any', [('state', '=', 'posted')])` |

### Hierarchical Operators
| Operator | Description | Example |
|----------|-------------|---------|
| `child_of` | Record or descendants | `('category_id', 'child_of', [5])` |
| `parent_of` | Record or ancestors | `('id', 'parent_of', [100])` |

## Boolean Logic

### AND (Default)
```python
# These are equivalent
[('a', '=', 1), ('b', '=', 2)]
['&', ('a', '=', 1), ('b', '=', 2)]
```

### OR
```python
# a OR b
['|', ('a', '=', 1), ('b', '=', 2)]

# a OR b OR c (nested)
['|', '|', ('a', '=', 1), ('b', '=', 2), ('c', '=', 3)]
```

### NOT
```python
# NOT a
['!', ('a', '=', 1)]

# NOT (a AND b)
['!', '&', ('a', '=', 1), ('b', '=', 2)]
```

### Complex Expressions
```python
# (a AND b) OR c
['|', '&', ('a', '=', 1), ('b', '=', 2), ('c', '=', 3)]

# a AND (b OR c)
['&', ('a', '=', 1), '|', ('b', '=', 2), ('c', '=', 3)]

# NOT a AND (b OR c)
['&', '!', ('a', '=', 1), '|', ('b', '=', 2), ('c', '=', 3)]
```

## Field Traversal

### Dot Notation
```python
# Access related field
[('partner_id.country_id.code', '=', 'US')]

# Multiple levels
[('order_id.partner_id.commercial_partner_id.is_company', '=', True)]
```

### Using `any` Operator
```python
# Records with at least one matching related record
[('line_ids', 'any', [('product_id.type', '=', 'service')])]

# Equivalent to dot notation for simple cases
[('partner_id', 'any', [('country_id.code', '=', 'US')])]
```

## Special Values

### Context Variables
```python
# Current user
[('user_id', '=', uid)]

# Current company
[('company_id', '=', company_id)]

# Allowed companies
[('company_id', 'in', company_ids)]

# Today's date
[('date', '=', context_today())]
```

### Boolean Fields
```python
# Active records
[('active', '=', True)]

# Archived records
[('active', '=', False)]
```

### Empty/Null Values
```python
# Field is empty
[('partner_id', '=', False)]

# Field is not empty
[('partner_id', '!=', False)]
```

## Search Method

### Basic Search
```python
records = self.env['res.partner'].search([
    ('is_company', '=', True),
    ('country_id.code', '=', 'US')
], limit=10, order='name')
```

### Search with Count
```python
count = self.env['sale.order'].search_count([
    ('state', '=', 'draft')
])
```

### Search Read (Optimized)
```python
data = self.env['res.partner'].search_read(
    [('is_company', '=', True)],
    fields=['name', 'email', 'phone'],
    limit=100,
    order='name'
)
```

## Search Views

### Field Filters
```xml
<search>
    <!-- Simple field search -->
    <field name="name"/>

    <!-- Custom filter domain -->
    <field name="name" string="Name/Code"
           filter_domain="['|', ('name', 'ilike', self), ('code', 'ilike', self)]"/>

    <!-- Hierarchical search -->
    <field name="category_id" filter_domain="[('category_id', 'child_of', self)]"/>
</search>
```

### Predefined Filters
```xml
<search>
    <filter string="My Records" name="my" domain="[('user_id', '=', uid)]"/>
    <filter string="Draft" name="draft" domain="[('state', '=', 'draft')]"/>
    <filter string="This Month" name="this_month"
            domain="[('date', '>=', (context_today() - relativedelta(day=1)).strftime('%Y-%m-%d')),
                     ('date', '&lt;', (context_today() + relativedelta(months=1, day=1)).strftime('%Y-%m-%d'))]"/>
    <separator/>
    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
</search>
```

### Group By
```xml
<search>
    <group expand="0" string="Group By">
        <filter string="Partner" name="partner" context="{'group_by': 'partner_id'}"/>
        <filter string="Month" name="month" context="{'group_by': 'date:month'}"/>
    </group>
</search>
```

## Domain Optimization

### Efficient Patterns
```python
# Good: Use 'in' for multiple values
[('state', 'in', ['draft', 'sent', 'pending'])]

# Bad: Multiple OR conditions
['|', '|', ('state', '=', 'draft'), ('state', '=', 'sent'), ('state', '=', 'pending')]
```

### Index-Friendly
```python
# Good: Query on indexed fields first
[('company_id', '=', 1), ('state', '=', 'draft')]

# Use child_of for hierarchies (uses parent_path)
[('category_id', 'child_of', parent_id)]
```

## Key Files

| File | Purpose |
|------|---------|
| `odoo/orm/domains.py` | Domain AST system |
| `odoo/osv/expression.py` | Legacy expression (deprecated) |
| `odoo/orm/models.py` | search() method |

## Common Examples

### Active Records
```python
[('active', '=', True)]
```

### Date Range
```python
[('date', '>=', '2025-01-01'), ('date', '<=', '2025-12-31')]
```

### Multi-Company
```python
['|', ('company_id', '=', False), ('company_id', 'in', company_ids)]
```

### User's Records
```python
[('user_id', '=', uid)]
# or
[('create_uid', '=', uid)]
```

### Related Record Exists
```python
[('order_ids', '!=', False)]
```

### Text Search
```python
['|', ('name', 'ilike', search_term), ('email', 'ilike', search_term)]
```
