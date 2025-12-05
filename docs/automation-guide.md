# Odoo 19.0 Automation Guide

## Automation Mechanisms Overview

| Mechanism | Module | Use Case |
|-----------|--------|----------|
| Scheduled Actions | `base` (ir.cron) | Periodic tasks |
| Server Actions | `base` (ir.actions.server) | Triggered operations |
| Automated Actions | `base_automation` | Event-driven rules |
| Mail Activities | `mail` | Task/workflow tracking |

## Scheduled Actions (ir.cron)

### Creating a Cron Job

```xml
<record id="ir_cron_my_task" model="ir.cron">
    <field name="name">My Scheduled Task</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="state">code</field>
    <field name="code">model._cron_my_task()</field>
    <field name="interval_number">1</field>
    <field name="interval_type">days</field>
    <field name="numbercall">-1</field>
    <field name="active">True</field>
</record>
```

### Python Implementation
```python
class MyModel(models.Model):
    _name = 'my.model'

    @api.model
    def _cron_my_task(self):
        records = self.search([('state', '=', 'pending')])
        for record in records:
            record.process()
```

### Interval Types
| Type | Description |
|------|-------------|
| `minutes` | Every N minutes |
| `hours` | Every N hours |
| `days` | Every N days |
| `weeks` | Every N weeks |
| `months` | Every N months |

### Progress Tracking (Batch Processing)
```python
def _cron_process_large_batch(self):
    cron = self.env.ref('my_module.ir_cron_my_task')
    cron._add_progress()

    records = self.search([('processed', '=', False)], limit=100)
    for i, record in enumerate(records):
        record.process()
        remaining = cron._commit_progress(processed=i+1)
        if remaining <= 0:
            break  # Time limit reached
```

### Triggering Cron Jobs
```python
# Immediate trigger
cron._trigger()

# Scheduled trigger
from datetime import datetime, timedelta
cron._trigger(at=datetime.now() + timedelta(hours=1))
```

## Server Actions (ir.actions.server)

### Action Types
| State | Description |
|-------|-------------|
| `code` | Execute Python code |
| `object_create` | Create new record |
| `object_write` | Update current record |
| `object_copy` | Duplicate record |
| `multi` | Execute multiple actions |
| `webhook` | Send HTTP POST |

### Python Code Action
```xml
<record id="action_process_record" model="ir.actions.server">
    <field name="name">Process Record</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="state">code</field>
    <field name="code">
for record in records:
    record.write({'state': 'processed'})
    </field>
</record>
```

### Execution Context Variables
| Variable | Description |
|----------|-------------|
| `env` | Environment |
| `model` | Target model |
| `record` | Single active record |
| `records` | All active records |
| `user` | Current user |
| `datetime`, `time` | Time utilities |
| `UserError` | Exception class |
| `log` | Logging function |

## Automated Actions (base_automation)

### Trigger Types

#### Record Triggers
| Trigger | Fires When |
|---------|------------|
| `on_create` | Record created |
| `on_write` | Record modified |
| `on_create_or_write` | Created or modified |
| `on_unlink` | Record deleted |
| `on_archive` | Record archived |
| `on_unarchive` | Record unarchived |

#### Field-Specific Triggers
| Trigger | Fires When |
|---------|------------|
| `on_stage_set` | Stage changed |
| `on_state_set` | State changed |
| `on_tag_set` | Tags modified |
| `on_user_set` | User assigned |
| `on_priority_set` | Priority changed |

#### Time-Based Triggers
| Trigger | Fires When |
|---------|------------|
| `on_time` | Based on date field |
| `on_time_created` | After creation date |
| `on_time_updated` | After modification date |

#### Message Triggers
| Trigger | Fires When |
|---------|------------|
| `on_message_received` | Customer message received |
| `on_message_sent` | Message sent |

### Creating an Automation Rule
```xml
<record id="rule_auto_assign" model="base.automation">
    <field name="name">Auto-Assign to Team</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="trigger">on_create</field>
    <field name="filter_domain">[('team_id', '=', False)]</field>
    <field name="action_server_ids" eval="[(4, ref('action_assign_team'))]"/>
</record>
```

### Filter Domains
```xml
<!-- Pre-filter: Condition BEFORE change -->
<field name="filter_pre_domain">[('state', '=', 'draft')]</field>

<!-- Post-filter: Condition AFTER change -->
<field name="filter_domain">[('state', '=', 'confirmed')]</field>
```

### Time-Based Configuration
```xml
<record id="rule_reminder" model="base.automation">
    <field name="name">Send Reminder</field>
    <field name="trigger">on_time</field>
    <field name="trg_date_id" ref="field_my_model__deadline"/>
    <field name="trg_date_range">-1</field>
    <field name="trg_date_range_type">day</field>
</record>
```

## Mail Activities

### Activity Types
```xml
<record id="activity_type_review" model="mail.activity.type">
    <field name="name">Review</field>
    <field name="delay_count">2</field>
    <field name="delay_unit">days</field>
    <field name="icon">fa-eye</field>
    <field name="res_model">my.model</field>
</record>
```

### Creating Activities
```python
record.activity_schedule(
    'my_module.activity_type_review',
    user_id=reviewer.id,
    summary="Please review this document",
    date_deadline=fields.Date.today() + timedelta(days=3)
)
```

### Activity Chaining
```xml
<field name="triggered_next_type_id" ref="activity_type_approve"/>
<!-- OR -->
<field name="suggested_next_type_ids" eval="[(6, 0, [
    ref('activity_type_approve'),
    ref('activity_type_reject')
])]"/>
```

## Key Files

| File | Purpose |
|------|---------|
| `odoo/addons/base/models/ir_cron.py` | Cron framework |
| `odoo/addons/base/models/ir_actions.py` | Server actions |
| `addons/base_automation/models/base_automation.py` | Automation rules |
| `addons/mail/models/mail_activity.py` | Activities |

## Best Practices

1. **Use time-based triggers** for large datasets (vs. write triggers)
2. **Filter early** with `filter_pre_domain` and `filter_domain`
3. **Batch operations** with `_commit_progress()` for long-running crons
4. **Monitor failures** - crons deactivate after 5 failures in 7 days
5. **Keep code simple** - complex logic belongs in model methods
6. **Test automation rules** with different user permissions
7. **Log actions** via `log()` function for debugging
