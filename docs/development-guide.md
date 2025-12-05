# Odoo 19.0 Development Guide

## Prerequisites

### System Requirements
- **Python**: 3.10, 3.11, 3.12, or 3.13
- **PostgreSQL**: 12+ (13+ recommended)
- **Git**: For source control
- **Node.js**: Optional, for frontend development

### Operating System
- Ubuntu 24.04 LTS / Debian 12 (recommended)
- macOS 12+
- Windows 10/11 with WSL2

## Installation

### Quick Setup (Development)

```bash
# Clone the repository
git clone https://github.com/odoo/odoo.git
cd odoo

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # Linux/macOS
# or: venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt

# Setup PostgreSQL user
sudo -u postgres createuser -s $USER

# Create database
createdb odoo_dev

# Run Odoo
./odoo-bin -d odoo_dev -i base
```

### Debian Dependencies

```bash
# Install system dependencies
./setup/debinstall.sh

# Or manually
sudo apt-get install python3-dev libxml2-dev libxslt1-dev \
  libldap2-dev libsasl2-dev libjpeg-dev zlib1g-dev \
  libpq-dev postgresql postgresql-client
```

## Running Odoo

### Development Server

```bash
# Basic startup
./odoo-bin -d mydb -i base

# With specific addons path
./odoo-bin -d mydb --addons-path=addons,../my-addons

# Development mode (auto-reload)
./odoo-bin -d mydb --dev=all

# Specific port
./odoo-bin -d mydb --http-port=8070
```

### Development Mode Options

```bash
--dev=all           # Enable all dev features
--dev=reload        # Auto-reload on code changes
--dev=qweb          # QWeb template debugging
--dev=xml           # XML validation
```

### Interactive Shell

```bash
# Start Python shell with Odoo environment
./odoo-bin shell -d mydb

# In shell:
>>> env['res.partner'].search([])
>>> env['sale.order'].browse(1)
```

## CLI Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `server` | Start HTTP server | `./odoo-bin` |
| `shell` | Interactive shell | `./odoo-bin shell -d mydb` |
| `scaffold` | Create module | `./odoo-bin scaffold mymodule addons` |
| `db` | Database management | `./odoo-bin db list` |
| `cloc` | Count lines of code | `./odoo-bin cloc -d mydb` |
| `i18n` | Translation tools | `./odoo-bin i18n -d mydb` |
| `populate` | Generate demo data | `./odoo-bin populate -d mydb` |

## Creating a New Module

### Using Scaffold

```bash
./odoo-bin scaffold my_module addons
```

Creates:
```
addons/my_module/
├── __init__.py
├── __manifest__.py
├── models/__init__.py
├── models/models.py
├── views/views.xml
├── security/ir.model.access.csv
├── controllers/__init__.py
├── controllers/controllers.py
├── static/description/icon.png
└── demo/demo.xml
```

### Manifest File (`__manifest__.py`)

```python
{
    'name': 'My Module',
    'version': '19.0.1.0.0',
    'summary': 'Short description',
    'description': """Long description""",
    'author': 'Your Name',
    'website': 'https://example.com',
    'license': 'LGPL-3',
    'category': 'Uncategorized',
    'depends': ['base', 'sale'],
    'data': [
        'security/ir.model.access.csv',
        'views/views.xml',
    ],
    'demo': [
        'demo/demo.xml',
    ],
    'installable': True,
    'application': True,
    'auto_install': False,
}
```

### Creating a Model

```python
# models/my_model.py
from odoo import models, fields, api

class MyModel(models.Model):
    _name = 'my.model'
    _description = 'My Model'

    name = fields.Char(string='Name', required=True)
    description = fields.Text(string='Description')
    state = fields.Selection([
        ('draft', 'Draft'),
        ('confirmed', 'Confirmed'),
    ], default='draft')
    partner_id = fields.Many2one('res.partner', string='Partner')
    line_ids = fields.One2many('my.model.line', 'model_id', string='Lines')

    @api.depends('line_ids.amount')
    def _compute_total(self):
        for record in self:
            record.total = sum(record.line_ids.mapped('amount'))

    total = fields.Float(compute='_compute_total', store=True)
```

### Creating Views

```xml
<!-- views/views.xml -->
<odoo>
    <!-- Tree View -->
    <record id="my_model_tree" model="ir.ui.view">
        <field name="name">my.model.tree</field>
        <field name="model">my.model</field>
        <field name="arch" type="xml">
            <tree>
                <field name="name"/>
                <field name="partner_id"/>
                <field name="state"/>
                <field name="total"/>
            </tree>
        </field>
    </record>

    <!-- Form View -->
    <record id="my_model_form" model="ir.ui.view">
        <field name="name">my.model.form</field>
        <field name="model">my.model</field>
        <field name="arch" type="xml">
            <form>
                <header>
                    <button name="action_confirm" type="object"
                            string="Confirm" states="draft"/>
                    <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="partner_id"/>
                    </group>
                    <notebook>
                        <page string="Lines">
                            <field name="line_ids"/>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <!-- Action -->
    <record id="my_model_action" model="ir.actions.act_window">
        <field name="name">My Models</field>
        <field name="res_model">my.model</field>
        <field name="view_mode">tree,form</field>
    </record>

    <!-- Menu -->
    <menuitem id="my_model_menu" name="My Module"
              action="my_model_action" sequence="10"/>
</odoo>
```

### Creating a Controller

```python
# controllers/main.py
from odoo import http
from odoo.http import request

class MyController(http.Controller):

    @http.route('/my_module/data', type='json', auth='user')
    def get_data(self, **kwargs):
        records = request.env['my.model'].search([])
        return records.read(['name', 'state'])

    @http.route('/my_module/page', type='http', auth='public', website=True)
    def my_page(self, **kwargs):
        return request.render('my_module.my_template', {
            'records': request.env['my.model'].sudo().search([])
        })
```

## Testing

### Running Tests

```bash
# Run all tests for a module
./odoo-bin -d test_db --test-enable --stop-after-init -i my_module

# Run specific test class
./odoo-bin -d test_db --test-enable --stop-after-init \
  --test-tags=/my_module:TestMyModel

# Run with coverage
coverage run ./odoo-bin -d test_db --test-enable -i my_module
coverage report
```

### Writing Tests

```python
# tests/test_my_model.py
from odoo.tests import TransactionCase, tagged

@tagged('post_install', '-at_install')
class TestMyModel(TransactionCase):

    def setUp(self):
        super().setUp()
        self.partner = self.env['res.partner'].create({
            'name': 'Test Partner'
        })

    def test_create_record(self):
        record = self.env['my.model'].create({
            'name': 'Test Record',
            'partner_id': self.partner.id,
        })
        self.assertEqual(record.state, 'draft')

    def test_confirm_action(self):
        record = self.env['my.model'].create({'name': 'Test'})
        record.action_confirm()
        self.assertEqual(record.state, 'confirmed')
```

## Database Operations

```bash
# List databases
./odoo-bin db list

# Create database
./odoo-bin db create mydb

# Drop database
./odoo-bin db drop mydb

# Backup database
pg_dump mydb > backup.sql

# Restore database
psql mydb < backup.sql
```

## Module Operations

```bash
# Install module
./odoo-bin -d mydb -i my_module

# Update module
./odoo-bin -d mydb -u my_module

# Update all modules
./odoo-bin -d mydb -u all
```

## Debugging

### Debug Logging

```bash
# Verbose logging
./odoo-bin -d mydb --log-level=debug

# Log specific modules
./odoo-bin -d mydb --log-handler=odoo.addons.my_module:DEBUG
```

### Python Debugger

```python
# In your code
import pdb; pdb.set_trace()

# Or with ipdb
import ipdb; ipdb.set_trace()
```

## Frontend Development

### JavaScript/OWL Components

Located in `static/src/js/`:

```javascript
/** @odoo-module */
import { Component } from "@odoo/owl";
import { registry } from "@web/core/registry";

class MyComponent extends Component {
    static template = "my_module.MyComponent";

    setup() {
        // Component setup
    }
}

registry.category("actions").add("my_action", MyComponent);
```

### SCSS Styles

Located in `static/src/scss/`:

```scss
// my_styles.scss
.my-component {
    .header {
        background: $o-brand-primary;
    }
}
```

### Asset Bundles

In `__manifest__.py`:

```python
'assets': {
    'web.assets_backend': [
        'my_module/static/src/js/**/*',
        'my_module/static/src/scss/**/*',
    ],
},
```

## Performance Tips

1. **Use `sudo()`** sparingly - bypasses security
2. **Batch operations** - use `create()` with list of values
3. **Prefetch fields** - include related fields in search
4. **Use `store=True`** for frequently read computed fields
5. **Index important fields** - `index=True` for search fields

## Common Patterns

### Computed Fields

```python
total = fields.Float(compute='_compute_total', store=True)

@api.depends('line_ids.amount')
def _compute_total(self):
    for record in self:
        record.total = sum(record.line_ids.mapped('amount'))
```

### Constraints

```python
@api.constrains('date_start', 'date_end')
def _check_dates(self):
    for record in self:
        if record.date_end < record.date_start:
            raise ValidationError("End date must be after start date")
```

### Onchange

```python
@api.onchange('partner_id')
def _onchange_partner(self):
    if self.partner_id:
        self.name = self.partner_id.name
```

## Resources

- **Official Docs**: https://www.odoo.com/documentation/19.0/
- **Developer Tutorials**: https://www.odoo.com/documentation/19.0/developer/howtos.html
- **API Reference**: https://www.odoo.com/documentation/19.0/developer/reference/
- **GitHub**: https://github.com/odoo/odoo
