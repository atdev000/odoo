# Odoo 19.0 Testing Guide

## Test Base Classes

| Class | Purpose | Use Case |
|-------|---------|----------|
| `TransactionCase` | Per-method savepoint rollback | Standard unit/integration tests |
| `SingleTransactionCase` | Shared transaction | Performance-critical tests |
| `HttpCase` | Browser automation | UI/integration tests, tours |
| `BaseCase` | Abstract base | All Odoo tests inherit from this |

## Running Tests

### Command Line

```bash
# Run tests for a module
./odoo-bin -d testdb --test-enable -i my_module --stop-after-init

# Run specific test tags
./odoo-bin -d testdb --test-enable --test-tags "post_install"

# Run specific test class
./odoo-bin -d testdb --test-enable --test-tags ":TestMyClass"

# Run specific test method
./odoo-bin -d testdb --test-enable --test-tags ":TestMyClass.test_method"

# Run tests for specific module
./odoo-bin -d testdb --test-enable --test-tags "/my_module"
```

### Test Tags

| Tag | Description |
|-----|-------------|
| `standard` | Run at module installation (default) |
| `at_install` | Run during module installation |
| `post_install` | Run after module installation |
| `-tag_name` | Remove inherited tag |
| `external` | Tests requiring external services |

## Writing Tests

### Basic Test Structure

```python
from odoo.tests import TransactionCase, tagged

@tagged('post_install', '-at_install')
class TestMyModel(TransactionCase):

    @classmethod
    def setUpClass(cls):
        """Run once per test class"""
        super().setUpClass()
        cls.partner = cls.env['res.partner'].create({
            'name': 'Test Partner'
        })

    def setUp(self):
        """Run before each test method"""
        super().setUp()

    def test_create_record(self):
        """Test method name must start with 'test_'"""
        record = self.env['my.model'].create({
            'name': 'Test Record',
            'partner_id': self.partner.id,
        })
        self.assertEqual(record.state, 'draft')

    def test_workflow(self):
        """Test a complete workflow"""
        record = self.env['my.model'].create({'name': 'Test'})
        record.action_confirm()
        self.assertEqual(record.state, 'confirmed')
```

### Common Assertions

```python
# Standard assertions
self.assertEqual(expected, actual)
self.assertTrue(condition)
self.assertFalse(condition)
self.assertIn(item, container)
self.assertNotIn(item, container)

# Exception testing
with self.assertRaises(ValidationError):
    record.write({'invalid': 'data'})

# Odoo-specific assertions
self.assertRecordValues(records, [
    {'field1': value1, 'field2': value2},
])
```

### HTTP/UI Tests

```python
from odoo.tests import HttpCase, tagged

@tagged('post_install', '-at_install')
class TestUI(HttpCase):

    def test_tour(self):
        """Run a JavaScript tour"""
        self.start_tour("/web", 'my_tour', login="admin")

    def test_http_request(self):
        """Test HTTP endpoint"""
        response = self.url_open('/my/endpoint')
        self.assertEqual(response.status_code, 200)
```

### Test Helpers

```python
from odoo.tests.common import new_test_user
from freezegun import freeze_time

@tagged('post_install', '-at_install')
class TestHelpers(TransactionCase):

    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # Create test user with specific groups
        cls.user = new_test_user(
            cls.env,
            login='testuser',
            groups='base.group_user,sales_team.group_sale_manager'
        )

    @freeze_time('2024-01-01')
    def test_date_logic(self):
        """Test with frozen time"""
        record = self.env['my.model'].create({'name': 'Test'})
        self.assertEqual(record.date, '2024-01-01')
```

## Test File Organization

```
addons/my_module/tests/
├── __init__.py          # Import all test modules
├── common.py            # Shared test setup classes
├── test_model_a.py      # Tests for model A
├── test_model_b.py      # Tests for model B
└── test_workflow.py     # Integration/workflow tests
```

### __init__.py Example

```python
from . import common
from . import test_model_a
from . import test_model_b
from . import test_workflow
```

## Best Practices

### Do's

1. **Use `setUpClass`** for expensive setup (creating records)
2. **Use descriptive test names** that explain what's being tested
3. **Test one thing per test** method
4. **Use `@tagged`** to control when tests run
5. **Clean up external resources** in `tearDown`

### Don'ts

1. **Don't commit/rollback** explicitly (framework handles this)
2. **Don't make external HTTP calls** (blocked by default)
3. **Don't depend on demo data** unless explicitly loaded
4. **Don't skip proper assertions** - always verify expected behavior

## Coverage

```bash
# Run with coverage
coverage run ./odoo-bin -d testdb --test-enable -i my_module --stop-after-init

# Generate report
coverage report -m

# Generate HTML report
coverage html
```

## Debugging Tests

```python
# Add breakpoint in test
import pdb; pdb.set_trace()

# Or with ipdb (if installed)
import ipdb; ipdb.set_trace()

# Run with verbose output
./odoo-bin -d testdb --test-enable -i my_module --log-level=debug
```

## Common Test Patterns

### Testing Computed Fields

```python
def test_computed_total(self):
    order = self.env['sale.order'].create({
        'partner_id': self.partner.id,
        'order_line': [(0, 0, {
            'product_id': self.product.id,
            'product_uom_qty': 2,
            'price_unit': 100,
        })]
    })
    self.assertEqual(order.amount_total, 200.0)
```

### Testing Constraints

```python
def test_date_constraint(self):
    with self.assertRaises(ValidationError):
        self.env['my.model'].create({
            'date_start': '2024-01-10',
            'date_end': '2024-01-01',  # End before start
        })
```

### Testing Access Rights

```python
def test_access_rights(self):
    record = self.env['my.model'].create({'name': 'Test'})

    # Switch to limited user
    record_as_user = record.with_user(self.limited_user)

    with self.assertRaises(AccessError):
        record_as_user.unlink()
```
