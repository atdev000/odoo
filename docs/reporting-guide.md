# Odoo 19.0 Reporting Guide

## Report Architecture

```
HTTP Request → ReportController → IrActionsReport._render()
                                        ↓
                    _render_qweb_html | _render_qweb_pdf | _render_qweb_text
                                        ↓
                              QWeb Template Engine
                                        ↓
                              Output (HTML/PDF/Text)
```

## Report Types

| Type | Output | Use Case |
|------|--------|----------|
| `qweb-pdf` | PDF via wkhtmltopdf | Printable documents |
| `qweb-html` | HTML | Screen display |
| `qweb-text` | Plain text | Data export |

## Creating a Report

### 1. Report Action Definition
```xml
<record id="action_report_myreport" model="ir.actions.report">
    <field name="name">My Report</field>
    <field name="model">my.model</field>
    <field name="report_type">qweb-pdf</field>
    <field name="report_name">my_module.report_myreport</field>
    <field name="print_report_name">'MyReport_%s' % object.name</field>
    <field name="paperformat_id" ref="paperformat_euro"/>
</record>
```

### 2. QWeb Template
```xml
<template id="report_myreport">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="o">
            <t t-call="web.external_layout">
                <div class="page">
                    <h1 t-field="o.name"/>
                    <table class="table table-sm">
                        <thead>
                            <tr><th>Field</th><th>Value</th></tr>
                        </thead>
                        <tbody>
                            <tr t-foreach="o.line_ids" t-as="line">
                                <td t-field="line.name"/>
                                <td t-field="line.amount"/>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </t>
        </t>
    </t>
</template>
```

## Layout Templates

### Available Layouts
| Layout | Purpose |
|--------|---------|
| `web.html_container` | Base HTML structure |
| `web.external_layout` | Company header/footer |
| `web.internal_layout` | Internal documents |
| `web.basic_layout` | Minimal container |

### External Layout Variants
- `external_layout_standard` - Classic style
- `external_layout_bold` - Bold headers
- `external_layout_boxed` - Boxed sections
- `external_layout_striped` - Striped tables

## Custom Data Model

```python
class MyReportHandler(models.Model):
    _name = 'report.my_module.report_myreport'
    _description = 'My Report Handler'

    @api.model
    def _get_report_values(self, docids, data=None):
        docs = self.env['my.model'].browse(docids)
        return {
            'doc_ids': docids,
            'doc_model': 'my.model',
            'docs': docs,
            'computed_data': self._compute_totals(docs),
        }

    def _compute_totals(self, docs):
        return sum(doc.amount for doc in docs)
```

## Paper Formats

### Predefined Formats
```xml
<record id="paperformat_custom" model="report.paperformat">
    <field name="name">Custom A4</field>
    <field name="format">A4</field>
    <field name="orientation">Portrait</field>
    <field name="margin_top">40</field>
    <field name="margin_bottom">28</field>
    <field name="margin_left">7</field>
    <field name="margin_right">7</field>
    <field name="header_spacing">35</field>
    <field name="dpi">90</field>
</record>
```

### Override via HTML Attributes
```html
<html data-report-landscape="1"
      data-report-dpi="96"
      data-report-margin-top="20"
      data-report-margin-bottom="20">
```

## PDF Generation (wkhtmltopdf)

### Requirements
- wkhtmltopdf with patched Qt (0.12.5+)
- Supports JavaScript rendering
- Header/footer via separate HTML files

### Multi-Document Handling
```xml
<!-- Each article becomes separate PDF section -->
<article t-att-data-oe-model="o._name"
         t-att-data-oe-id="o.id">
    <!-- Document content -->
</article>
```

## Report Caching

### Attachment Storage
```xml
<field name="attachment">
    'Report_%(object.name)s.pdf'
</field>
<field name="attachment_use" eval="True"/>
```

Reports are cached as `ir.attachment` records when `attachment` field is set.

## HTTP Routes

### Report URLs
```
/report/<converter>/<reportname>/<docids>
```
- `converter`: `html`, `pdf`, `text`
- `reportname`: Template XML ID
- `docids`: Comma-separated record IDs

### Download Route
```
/report/download?data=<json>&token=<token>
```

## Barcode Generation

```python
# In template
<img t-att-src="'/report/barcode/Code128/%s' % o.barcode"/>

# Supported types
# QR, Code128, EAN13, EAN8, UPCA, Code39
```

## Excel/CSV Export

### Export Controller
```python
# Routes
/web/export/csv
/web/export/xlsx

# Parameters
- model: Model name
- fields: List of field specifications
- domain: Filter domain
- grouped_by: Grouping fields
```

### Field Specification
```python
fields = [
    {'name': 'name', 'label': 'Name'},
    {'name': 'partner_id.name', 'label': 'Partner'},
    {'name': 'amount', 'label': 'Amount', 'aggregator': 'sum'},
]
```

## Key Files

| File | Purpose |
|------|---------|
| `odoo/addons/base/models/ir_actions_report.py` | Report engine |
| `addons/web/controllers/report.py` | HTTP routes |
| `addons/web/controllers/export.py` | CSV/XLSX export |
| `odoo/addons/base/models/ir_qweb.py` | QWeb engine |

## Best Practices

1. **Use `external_layout`** for consistent branding
2. **Split large tables** to avoid wkhtmltopdf slowdowns
3. **Cache reports** with attachment field for repeated access
4. **Test PDF rendering** - wkhtmltopdf differs from browser
5. **Use `t-field`** for proper formatting and translation
6. **Include page breaks** with CSS `page-break-before/after`
