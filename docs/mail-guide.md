# Odoo 19.0 Mail & Messaging Guide

## Making Models Messageable

### Basic Implementation
```python
class MyModel(models.Model):
    _name = 'my.model'
    _inherit = ['mail.thread', 'mail.activity.mixin']

    name = fields.Char(tracking=True)  # Track changes
    state = fields.Selection([...], tracking=True)
```

### Inherited Fields (Automatic)
- `message_ids` - All messages on record
- `message_follower_ids` - Followers subscribed
- `message_is_follower` - Current user is follower
- `activity_ids` - Scheduled activities
- `activity_state` - Activity status (overdue/today/planned)

## Posting Messages

### Basic Message Post
```python
record.message_post(
    body="<p>Message content</p>",
    subject="Optional Subject",
    message_type='comment',  # or 'notification', 'email'
    subtype_xmlid='mail.mt_comment',
    partner_ids=[partner.id],  # Notify specific partners
    attachment_ids=[att.id],   # Attach files
)
```

### Message Types
| Type | Description |
|------|-------------|
| `comment` | User comment (chatter) |
| `notification` | System notification |
| `email` | Email message |
| `auto_comment` | Automated comment |
| `user_notification` | Direct user notification |

### Tracking Field Changes
```python
# Automatic tracking
name = fields.Char(tracking=True)

# Custom tracking
@api.onchange('important_field')
def _track_important_change(self):
    self._track_set_log_message('<p>Important field changed</p>')
```

## Email Templates

### Creating Templates
```xml
<record id="email_template_example" model="mail.template">
    <field name="name">Example Template</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="subject">{{ object.name }} - Notification</field>
    <field name="body_html"><![CDATA[
        <p>Dear {{ object.partner_id.name }},</p>
        <p>Your record {{ object.name }} has been updated.</p>
        <p>Amount: {{ format_amount(object.amount, object.currency_id) }}</p>
    ]]></field>
    <field name="email_from">{{ (object.user_id.email or user.email) }}</field>
    <field name="email_to">{{ object.partner_id.email }}</field>
    <field name="report_template_ids" eval="[(4, ref('module.report_id'))]"/>
</record>
```

### Sending Templates
```python
template = self.env.ref('module.email_template_example')
template.send_mail(record.id, force_send=True)

# Batch sending
template.send_mail_batch([id1, id2, id3])
```

### Template Placeholders
| Placeholder | Description |
|-------------|-------------|
| `{{ object }}` | Current record |
| `{{ user }}` | Current user |
| `{{ ctx }}` | Context dictionary |
| `{{ format_date(date) }}` | Formatted date |
| `{{ format_amount(amt, cur) }}` | Formatted currency |

## Followers & Subscriptions

### Managing Followers
```python
# Subscribe partners
record.message_subscribe(
    partner_ids=[partner.id],
    subtype_ids=[subtype.id]  # Optional specific subtypes
)

# Unsubscribe
record.message_unsubscribe(partner_ids=[partner.id])

# Auto-subscribe on create
def _message_auto_subscribe_followers(self, updated_values, subtype_ids):
    return [
        (partner_id, subtype_ids, False)  # (partner, subtypes, reason)
    ]
```

### Message Subtypes
```xml
<record id="mt_order_confirmed" model="mail.message.subtype">
    <field name="name">Order Confirmed</field>
    <field name="res_model">sale.order</field>
    <field name="default" eval="True"/>
    <field name="description">Order has been confirmed</field>
</record>
```

## Mail Gateway (Incoming Email)

### Mail Alias Setup
```xml
<record id="alias_support" model="mail.alias">
    <field name="alias_name">support</field>
    <field name="alias_model_id" ref="model_helpdesk_ticket"/>
    <field name="alias_defaults">{'team_id': 1}</field>
    <field name="alias_contact">everyone</field>
</record>
```

### Processing Incoming Mail
```python
class MyModel(models.Model):
    _inherit = ['mail.thread', 'mail.alias.mixin']

    def message_new(self, msg_dict, custom_values=None):
        """Create new record from incoming email"""
        values = {
            'name': msg_dict.get('subject'),
            'email_from': msg_dict.get('email_from'),
            'description': msg_dict.get('body'),
        }
        values.update(custom_values or {})
        return super().create(values)

    def message_update(self, msg_dict, update_vals=None):
        """Update existing record from reply email"""
        if msg_dict.get('body'):
            self.write({'last_update': fields.Datetime.now()})
        return super().message_update(msg_dict, update_vals)
```

### Alias Contact Options
| Option | Description |
|--------|-------------|
| `everyone` | Anyone can email |
| `partners` | Only known partners |
| `followers` | Only followers |
| `employees` | Only internal users |

## Activities

### Activity Types
```xml
<record id="activity_review" model="mail.activity.type">
    <field name="name">Review Document</field>
    <field name="icon">fa-eye</field>
    <field name="delay_count">3</field>
    <field name="delay_unit">days</field>
    <field name="res_model">my.model</field>
</record>
```

### Scheduling Activities
```python
record.activity_schedule(
    'module.activity_review',
    user_id=reviewer.id,
    summary="Please review",
    date_deadline=fields.Date.today() + timedelta(days=3)
)

# Mark as done
record.activity_feedback(['module.activity_review'])

# Unlink activity
record.activity_unlink(['module.activity_review'])
```

## Discuss Channels

### Creating Channels
```python
channel = self.env['discuss.channel'].create({
    'name': 'Project Discussion',
    'channel_type': 'group',  # 'chat', 'channel', 'group'
})

# Add members
channel.add_members(partner_ids=[partner.id])

# Post to channel
channel.message_post(body="Hello team!")
```

## Real-Time Notifications

### Bus Integration
```python
# Send notification to user
self.env['bus.bus']._sendone(
    (self._cr.dbname, 'res.users', user_id),
    'notification',
    {'message': 'You have a new message'}
)
```

## Key Files

| File | Purpose |
|------|---------|
| `addons/mail/models/mail_thread.py` | Core mixin |
| `addons/mail/models/mail_message.py` | Message model |
| `addons/mail/models/mail_template.py` | Email templates |
| `addons/mail/models/mail_followers.py` | Follower system |
| `addons/mail/models/mail_activity.py` | Activities |

## Best Practices

1. **Always inherit `mail.thread`** for discussable models
2. **Use `tracking=True`** on important fields
3. **Define subtypes** for different notification scenarios
4. **Use templates** for consistent email formatting
5. **Set `_mail_post_access`** to control who can post
6. **Override `message_new`** for email-driven workflows
7. **Batch email sending** with `force_send=False`
