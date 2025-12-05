# Odoo 19.0 Performance Guide

## Caching System

### ORM Cache Decorator

```python
from odoo.tools import ormcache

class MyModel(models.Model):
    _name = 'my.model'

    @ormcache('self.id', 'mode')
    def get_computed_value(self, mode):
        # Expensive computation cached by id and mode
        return self._heavy_calculation(mode)
```

### Cache Key Expressions
```python
# Simple keys
@ormcache('key')
def get_param(self, key): ...

# Multiple keys
@ormcache('self.id', 'self.env.uid', 'field_name')
def get_field_value(self, field_name): ...

# Tuple keys
@ormcache('tuple(self.env.companies.ids)')
def get_company_data(self): ...
```

### Named Caches
| Cache | Size | Purpose |
|-------|------|---------|
| `default` | 8192 | General caching |
| `stable` | 1024 | Long-lived data |
| `assets` | 512 | Asset bundles |
| `templates` | 1024 | QWeb templates |
| `routing` | 1024 | URL routing |

```python
@ormcache('key', cache='stable')
def get_config(self, key):
    # Cached in 'stable' cache
    return self.env['ir.config_parameter'].get_param(key)
```

### Cache Invalidation
```python
# Clear specific method cache
self.env.registry.clear_cache()

# Clear all caches
self.env.registry.clear_all_caches()
```

## Prefetching

### Automatic Prefetching
```python
# Efficient - single query for all partners
for partner in partners:
    print(partner.name)  # Prefetched in batch

# Disable prefetching when not needed
for partner in partners.with_prefetch([]):
    print(partner.name)  # Individual queries
```

### Prefetch Field Attribute
```python
class MyModel(models.Model):
    # Always prefetch this field
    important_field = fields.Char(prefetch=True)

    # Never prefetch (e.g., large binary)
    large_data = fields.Binary(prefetch=False)
```

### Context Control
```python
# Disable field prefetching
records.with_context(prefetch_fields=False)

# Enable translation prefetching
records.with_context(prefetch_langs=True)
```

## Query Optimization

### Efficient Patterns
```python
# Good: Filter in database
records = self.env['my.model'].search([('state', '=', 'active')])

# Bad: Filter in Python
records = self.env['my.model'].search([])
active_records = records.filtered(lambda r: r.state == 'active')
```

### Search Read
```python
# Efficient: Only fetch needed fields
data = self.env['res.partner'].search_read(
    [('is_company', '=', True)],
    fields=['name', 'email'],
    limit=100
)
```

### Batch Operations
```python
# Good: Single write
records.write({'state': 'done'})

# Bad: Multiple writes
for record in records:
    record.write({'state': 'done'})
```

### SQL for Complex Queries
```python
def get_statistics(self):
    self.env.cr.execute("""
        SELECT state, COUNT(*), SUM(amount)
        FROM my_model
        WHERE create_date >= %s
        GROUP BY state
    """, (start_date,))
    return self.env.cr.dictfetchall()
```

## Profiling Tools

### SQL Profiler
```python
from odoo.tools.profiler import Profiler

with Profiler(collectors=['sql']) as profiler:
    # Code to profile
    records = self.env['res.partner'].search([])
    for r in records:
        _ = r.invoice_ids

# View results
print(profiler.format())
```

### Async Stack Sampling
```python
with Profiler(collectors=['traces_async']) as profiler:
    # Samples execution stack at 1ms intervals
    heavy_operation()

# JSON output for analysis
profiler.json()
```

### Memory Profiler
```python
from odoo.tools.profiler import MemoryCollector

with Profiler(collectors=[MemoryCollector()]) as profiler:
    large_operation()

# Memory allocation traces
print(profiler.format())
```

### QWeb Template Profiler
```python
from odoo.tools.profiler import QwebCollector

with Profiler(collectors=[QwebCollector()]) as profiler:
    report._render_qweb_html(docids)
```

### Debug Mode
```bash
# Run with profiling enabled
./odoo-bin --dev=all -d mydb
```

### Cache Statistics
```bash
# Send signal to Odoo process
kill -SIGUSR1 <pid>   # Log cache stats
kill -SIGUSR2 <pid>   # Log cache stats with memory
```

## Computed Field Optimization

### Store Computed Fields
```python
class MyModel(models.Model):
    total = fields.Float(compute='_compute_total', store=True)

    @api.depends('line_ids.amount')
    def _compute_total(self):
        for record in self:
            record.total = sum(record.line_ids.mapped('amount'))
```

### Avoid N+1 Queries
```python
# Bad: N+1 queries
for order in orders:
    partner_name = order.partner_id.name

# Good: Prefetch in batch
orders.mapped('partner_id')  # Single query
for order in orders:
    partner_name = order.partner_id.name  # From cache
```

## Configuration Tuning

### Workers and Threads
```ini
# odoo.conf
workers = 4              # Number of worker processes
max_cron_threads = 2     # Cron worker threads
limit_memory_hard = 2684354560  # 2.5GB
limit_memory_soft = 2147483648  # 2GB
limit_time_cpu = 60      # CPU time limit
limit_time_real = 120    # Real time limit
```

### Database Connection Pool
```ini
db_maxconn = 64          # Max connections per worker
```

### Asset Optimization
```ini
# Production mode
dev_mode =               # Empty = production assets
```

## Key Files

| File | Purpose |
|------|---------|
| `odoo/tools/cache.py` | Cache decorators |
| `odoo/tools/lru.py` | LRU implementation |
| `odoo/tools/profiler.py` | Profiling tools |
| `odoo/tools/query.py` | Query builder |

## Performance Checklist

### Development
- [ ] Use `@ormcache` for expensive computations
- [ ] Store frequently-accessed computed fields
- [ ] Use `search_read` with field selection
- [ ] Batch write operations
- [ ] Profile slow operations with SQL collector

### Production
- [ ] Configure appropriate worker count
- [ ] Set memory limits
- [ ] Enable asset bundling (no dev mode)
- [ ] Monitor cache hit ratios
- [ ] Use reverse proxy caching for static assets

### Database
- [ ] Ensure proper indexes on filtered fields
- [ ] Analyze query plans for slow queries
- [ ] Vacuum and reindex periodically
- [ ] Monitor connection pool usage
