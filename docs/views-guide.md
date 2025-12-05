# Odoo 19.0 Views & UI Guide

## View Types

| Type | Purpose |
|------|---------|
| `form` | Record editing interface |
| `list/tree` | Tabular data display |
| `kanban` | Card-based visualization |
| `calendar` | Time-based events |
| `pivot` | Multi-dimensional analysis |
| `graph` | Charts and graphs |
| `search` | Filters and grouping |

## Form View

### Basic Structure
```xml
<form string="My Form">
    <header>
        <button name="action_confirm" type="object" string="Confirm"
                invisible="state != 'draft'" class="btn-primary"/>
        <field name="state" widget="statusbar"/>
    </header>
    <sheet>
        <div class="oe_button_box" name="button_box">
            <button class="oe_stat_button" icon="fa-list"
                    name="action_view_lines" type="object">
                <field name="line_count" widget="statinfo" string="Lines"/>
            </button>
        </div>
        <group>
            <group string="General">
                <field name="name"/>
                <field name="partner_id"/>
            </group>
            <group string="Details">
                <field name="date"/>
                <field name="amount" widget="monetary"/>
            </group>
        </group>
        <notebook>
            <page string="Lines" name="lines">
                <field name="line_ids">
                    <list editable="bottom">
                        <field name="product_id"/>
                        <field name="quantity"/>
                        <field name="price"/>
                    </list>
                </field>
            </page>
            <page string="Notes" name="notes">
                <field name="notes" placeholder="Add notes..."/>
            </page>
        </notebook>
    </sheet>
    <chatter/>
</form>
```

## List/Tree View

```xml
<list string="My List" editable="bottom" multi_edit="1"
      decoration-danger="state == 'overdue'"
      decoration-muted="active == False">
    <header>
        <button name="action_confirm" type="object" string="Confirm"/>
    </header>
    <field name="sequence" widget="handle"/>
    <field name="name"/>
    <field name="partner_id" widget="many2one_avatar"/>
    <field name="date"/>
    <field name="amount" sum="Total"/>
    <field name="state" widget="badge"
           decoration-success="state == 'done'"
           decoration-info="state == 'draft'"/>
    <field name="active" column_invisible="True"/>
</list>
```

### List Attributes
| Attribute | Description |
|-----------|-------------|
| `editable` | `"top"` or `"bottom"` for inline editing |
| `multi_edit` | Allow editing multiple rows |
| `decoration-*` | Row styling based on conditions |
| `default_order` | Default sort order |
| `limit` | Records per page |

## Kanban View

```xml
<kanban default_group_by="stage_id" class="o_kanban_small_column"
        quick_create="true" on_create="quick_create">
    <field name="color"/>
    <field name="priority"/>
    <progressbar field="kanban_state"
                 colors='{"normal": "muted", "done": "success", "blocked": "danger"}'/>
    <templates>
        <t t-name="card">
            <div t-attf-class="#{record.color.value ? 'oe_kanban_color_' + record.color.value : ''}">
                <div class="o_kanban_image">
                    <field name="partner_id" widget="many2one_avatar"/>
                </div>
                <div class="oe_kanban_details">
                    <strong><field name="name"/></strong>
                    <div><field name="partner_id"/></div>
                    <field name="tag_ids" widget="many2many_tags"
                           options="{'color_field': 'color'}"/>
                </div>
                <div class="o_kanban_record_bottom">
                    <field name="priority" widget="priority"/>
                    <field name="activity_ids" widget="kanban_activity"/>
                </div>
            </div>
        </t>
    </templates>
</kanban>
```

## Search View

```xml
<search string="Search">
    <field name="name" string="Name"
           filter_domain="['|', ('name', 'ilike', self), ('code', 'ilike', self)]"/>
    <field name="partner_id"/>
    <separator/>
    <filter string="My Records" name="my_records"
            domain="[('user_id', '=', uid)]"/>
    <filter string="Archived" name="inactive"
            domain="[('active', '=', False)]"/>
    <separator/>
    <filter string="Late" name="late"
            domain="[('date', '&lt;', context_today().strftime('%Y-%m-%d'))]"/>
    <group expand="0" string="Group By">
        <filter string="Partner" name="partner"
                context="{'group_by': 'partner_id'}"/>
        <filter string="Date" name="date"
                context="{'group_by': 'date:month'}"/>
    </group>
    <searchpanel>
        <field name="category_id" icon="fa-filter" enable_counters="1"/>
        <field name="state" select="multi"/>
    </searchpanel>
</search>
```

## View Inheritance

### XPath Positions
```xml
<record id="view_form_inherit" model="ir.ui.view">
    <field name="name">my.model.form.inherit</field>
    <field name="model">my.model</field>
    <field name="inherit_id" ref="module.view_form"/>
    <field name="arch" type="xml">
        <!-- Add after element -->
        <xpath expr="//field[@name='name']" position="after">
            <field name="new_field"/>
        </xpath>

        <!-- Add before element -->
        <xpath expr="//group[@name='details']" position="before">
            <group string="New Section">
                <field name="another_field"/>
            </group>
        </xpath>

        <!-- Replace element -->
        <xpath expr="//button[@name='old_action']" position="replace">
            <button name="new_action" type="object" string="New Action"/>
        </xpath>

        <!-- Modify attributes -->
        <xpath expr="//field[@name='amount']" position="attributes">
            <attribute name="invisible">state == 'draft'</attribute>
            <attribute name="required">True</attribute>
        </xpath>

        <!-- Add inside element -->
        <xpath expr="//notebook" position="inside">
            <page string="New Tab" name="new_tab">
                <field name="extra_info"/>
            </page>
        </xpath>

        <!-- Move element -->
        <xpath expr="//field[@name='move_me']" position="move">
            <xpath expr="//group[@name='target']" position="inside"/>
        </xpath>
    </field>
</record>
```

### Shorthand Syntax
```xml
<!-- Direct field reference -->
<field name="existing_field" position="after">
    <field name="new_field"/>
</field>
```

## Common Widgets

### Text & Input
| Widget | Field Type | Description |
|--------|------------|-------------|
| `char` | Char | Default text input |
| `text` | Text | Multi-line textarea |
| `html` | Html | Rich text editor |
| `email` | Char | Email with mailto link |
| `url` | Char | URL with clickable link |
| `phone` | Char | Phone with tel: link |

### Numbers & Money
| Widget | Field Type | Description |
|--------|------------|-------------|
| `integer` | Integer | Number input |
| `float` | Float | Decimal input |
| `monetary` | Float | Currency formatted |
| `percentage` | Float | Percentage display |
| `progressbar` | Float/Integer | Progress bar |

### Selection & Status
| Widget | Field Type | Description |
|--------|------------|-------------|
| `selection` | Selection | Dropdown |
| `radio` | Selection | Radio buttons |
| `badge` | Selection | Colored badge |
| `statusbar` | Selection | Status bar |
| `priority` | Selection | Star rating |

### Relations
| Widget | Field Type | Description |
|--------|------------|-------------|
| `many2one` | Many2one | Dropdown with search |
| `many2one_avatar` | Many2one | With image |
| `many2many_tags` | Many2many | Tag pills |
| `many2many_checkboxes` | Many2many | Checkboxes |
| `one2many` | One2many | Embedded list/form |

### Special
| Widget | Field Type | Description |
|--------|------------|-------------|
| `image` | Binary | Image display/upload |
| `binary` | Binary | File upload |
| `date` | Date | Date picker |
| `datetime` | Datetime | Date+time picker |
| `handle` | Integer | Drag handle for sorting |
| `color` | Integer | Color picker |

## Modifiers (Conditional Display)

```xml
<!-- Hide field conditionally -->
<field name="discount" invisible="state != 'draft'"/>

<!-- Make readonly conditionally -->
<field name="amount" readonly="state == 'done'"/>

<!-- Make required conditionally -->
<field name="partner_id" required="type == 'sale'"/>

<!-- Hide column in list -->
<field name="internal_ref" column_invisible="True"/>

<!-- Complex conditions -->
<field name="special_field"
       invisible="state not in ('draft', 'pending') or not is_admin"/>
```

## Key Files

| File | Purpose |
|------|---------|
| `odoo/addons/base/models/ir_ui_view.py` | View model |
| `addons/web/static/src/views/` | View JS components |
| `addons/web/views/webclient_templates.xml` | Web client views |

## Best Practices

1. **Name views meaningfully** - `model.view_type.description`
2. **Use `name` attribute** on groups/pages for inheritance
3. **Prefer `position="attributes"`** for minor changes
4. **Set view priority** appropriately (lower = higher priority)
5. **Use modifiers** instead of hiding fields permanently
6. **Group related fields** logically
7. **Add search filters** for common queries
8. **Test inheritance** order with multiple modules
