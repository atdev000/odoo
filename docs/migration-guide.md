# Odoo 19.0 Migration Guide

## Migration Script Structure

### Directory Layout
```
my_module/
├── migrations/
│   ├── 1.0/
│   │   ├── pre-migration.py
│   │   └── post-migration.py
│   ├── 1.1/
│   │   └── post-update_data.py
│   └── 0.0.0/
│       └── end-cleanup.py
└── __manifest__.py
```

### Script Naming Convention
| Prefix | Stage | Timing |
|--------|-------|--------|
| `pre-` | Pre-migrate | Before module loading |
| `post-` | Post-migrate | After module initialization |
| `end-` | End-migrate | After ALL modules processed |

## Migration Script Template

### Basic Structure
```python
from odoo import api, SUPERUSER_ID

def migrate(cr, version):
    """
    Migration from version X to Y.

    Args:
        cr: Database cursor
        version: Currently installed version string
    """
    if not version:
        return

    # Migration logic here
    pass
```

### Pre-Migration (Schema Changes)
```python
from odoo.tools import sql

def migrate(cr, version):
    # Direct SQL for schema modifications
    cr.execute("""
        ALTER TABLE my_table ADD COLUMN IF NOT EXISTS new_field VARCHAR
    """)

    # Using SQL utilities
    if sql.column_exists(cr, 'my_table', 'old_column'):
        cr.execute("UPDATE my_table SET new_field = old_column")
        sql.drop_column(cr, 'my_table', 'old_column')
```

### Post-Migration (Data Transformation)
```python
from odoo import api, SUPERUSER_ID

def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})

    # Use ORM for data updates
    records = env['my.model'].search([('state', '=', 'old_state')])
    records.write({'state': 'new_state'})

    # Complex data transformation
    for record in env['my.model'].search([]):
        record.computed_field = record._calculate_value()
```

### End-Migration (Cross-Module Cleanup)
```python
def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})

    # Final consistency checks
    orphan_records = env['my.model'].search([
        ('parent_id', '!=', False),
        ('parent_id.active', '=', False)
    ])
    orphan_records.write({'parent_id': False})
```

## Version Numbering

### Format
```
[odoo_version.]module_major.module_minor[.module_patch]
```

### Examples
| Version | Meaning |
|---------|---------|
| `1.0` | Module version 1.0 (any Odoo) |
| `1.2.3` | Module version 1.2.3 |
| `19.0.1.0` | Odoo 19 specific, module 1.0 |
| `0.0.0` | Always runs on upgrade |

### Manifest Version
```python
# __manifest__.py
{
    'name': 'My Module',
    'version': '19.0.1.2.0',  # Odoo.Major.Minor.Patch
    'depends': ['base'],
}
```

## Execution Order

```
1. PRE-MIGRATIONS (version order, alphabetical within version)
2. pre_init_hook (from manifest)
3. Module initialization (models, views, data)
4. POST-MIGRATIONS
5. post_init_hook (from manifest)
6. Tests
7. Module registration
8. END-MIGRATIONS (after ALL modules)
```

## Common Migration Patterns

### Rename Field
```python
def migrate(cr, version):
    # Pre-migration: Column rename
    if sql.column_exists(cr, 'my_table', 'old_name'):
        cr.execute("""
            ALTER TABLE my_table
            RENAME COLUMN old_name TO new_name
        """)
```

### Merge Models
```python
def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})

    # Copy data from old model to new
    cr.execute("""
        INSERT INTO new_table (name, value, create_uid, create_date)
        SELECT name, value, create_uid, create_date
        FROM old_table
    """)
```

### Update XML IDs
```python
def migrate(cr, version):
    cr.execute("""
        UPDATE ir_model_data
        SET module = 'new_module'
        WHERE module = 'old_module'
        AND name LIKE 'view_%'
    """)
```

### Data Type Change
```python
def migrate(cr, version):
    # Add new column
    cr.execute("""
        ALTER TABLE my_table ADD COLUMN amount_new NUMERIC
    """)

    # Convert data
    cr.execute("""
        UPDATE my_table SET amount_new = amount_old::NUMERIC
    """)

    # Drop old column (post-migration after model loads)
```

### Many2many Relation Change
```python
def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})

    for record in env['my.model'].search([]):
        # Convert old relation to new
        new_ids = [r.new_related_id.id for r in record.old_relation_ids]
        record.write({'new_relation_ids': [(6, 0, new_ids)]})
```

## Module Hooks

### __manifest__.py
```python
{
    'pre_init_hook': 'pre_init_hook',
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
}
```

### __init__.py
```python
def pre_init_hook(env):
    """Called before module installation."""
    pass

def post_init_hook(env):
    """Called after module installation."""
    # Load default data
    env['my.model'].load_defaults()

def uninstall_hook(env):
    """Called before module uninstallation."""
    # Cleanup external resources
    pass
```

## SQL Utilities

### odoo.tools.sql
```python
from odoo.tools import sql

# Check existence
sql.table_exists(cr, 'table_name')
sql.column_exists(cr, 'table', 'column')
sql.index_exists(cr, 'index_name')

# Get metadata
columns = sql.table_columns(cr, 'table_name')

# Modify schema
sql.rename_column(cr, 'table', 'old_col', 'new_col')
sql.drop_column(cr, 'table', 'column')
sql.create_index(cr, 'index_name', 'table', ['col1', 'col2'])
sql.drop_index(cr, 'index_name', 'table')

# Constraints
sql.drop_constraint(cr, 'table', 'constraint_name')
sql.add_foreign_key(cr, 'table', 'col', 'ref_table', 'ref_col', 'cascade')
```

## Key Files

| File | Purpose |
|------|---------|
| `odoo/modules/migration.py` | Migration manager |
| `odoo/modules/loading.py` | Module loading orchestration |
| `odoo/tools/sql.py` | SQL utilities |

## Best Practices

1. **Test migrations** on a copy of production data
2. **Keep migrations idempotent** - safe to re-run
3. **Use transactions** - migrations run in single transaction
4. **Log progress** for long-running migrations
5. **Handle missing data** gracefully
6. **Version appropriately** - use `0.0.0` sparingly
7. **Document changes** in migration comments
8. **Backup before upgrading** production systems

## Troubleshooting

### Force Migration Re-run
```python
# In shell
registry = self.env.registry
registry._force_upgrade_scripts.add('my_module')
```

### Debug Migration
```python
def migrate(cr, version):
    import logging
    _logger = logging.getLogger(__name__)

    _logger.info("Starting migration from %s", version)
    # ... migration code ...
    _logger.info("Migration complete")
```

### Check Installed Version
```sql
SELECT latest_version FROM ir_module_module WHERE name = 'my_module';
```
