# Odoo 19.0 Assets & Bundling Guide

## Asset Bundle Architecture

### Main Bundles
| Bundle | Purpose |
|--------|---------|
| `web.assets_backend` | Full backend UI |
| `web.assets_frontend` | Portal/website UI |
| `web.assets_frontend_minimal` | Minimal frontend runtime |
| `web.assets_tests` | Test suite assets |

## Adding Assets to Bundles

### In Module Manifest
```python
# __manifest__.py
{
    'name': 'My Module',
    'assets': {
        'web.assets_backend': [
            # Add files
            'my_module/static/src/js/my_component.js',
            'my_module/static/src/scss/my_styles.scss',
            'my_module/static/src/xml/my_templates.xml',

            # Include another bundle
            ('include', 'my_module._assets_helpers'),

            # Insert after specific file
            ('after', 'web/static/src/core/utils.js',
             'my_module/static/src/core/my_utils.js'),

            # Insert before
            ('before', 'web/static/src/views/view.js',
             'my_module/static/src/views/my_view.js'),

            # Replace file
            ('replace', 'other_module/static/src/old.js',
             'my_module/static/src/new.js'),

            # Remove file
            ('remove', 'unwanted_module/static/src/file.js'),

            # Glob patterns
            'my_module/static/src/components/**/*',
        ],
        'web.assets_tests': [
            'my_module/static/tests/**/*',
        ],
    },
}
```

## JavaScript Module System

### ES6 Module Structure
```javascript
/** @odoo-module **/

import { Component, useState } from "@odoo/owl";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";

export class MyComponent extends Component {
    static template = "my_module.MyTemplate";
    static props = {
        record: Object,
        optional: { type: String, optional: true },
    };

    setup() {
        this.state = useState({ count: 0 });
        this.orm = useService("orm");
        this.notification = useService("notification");
    }

    async onClick() {
        await this.orm.call("my.model", "my_method", [this.props.record.id]);
        this.notification.add("Success!", { type: "success" });
    }
}

// Register component
registry.category("fields").add("my_widget", {
    component: MyComponent,
});
```

### Module Path Resolution
```
File: my_module/static/src/components/my_component.js
Module: @my_module/components/my_component

Import: import { MyComponent } from "@my_module/components/my_component";
```

### Common Imports
```javascript
// OWL framework
import { Component, useState, useEffect, useRef } from "@odoo/owl";

// Core utilities
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { _t } from "@web/core/l10n/translation";

// Fields and views
import { standardFieldProps } from "@web/views/fields/standard_field_props";
import { useRecordObserver } from "@web/model/relational_model/utils";
```

## XML Templates

### QWeb Templates for OWL
```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="my_module.MyTemplate">
        <div class="my-component">
            <h3 t-esc="props.title"/>
            <button t-on-click="onClick" class="btn btn-primary">
                Click (<t t-esc="state.count"/>)
            </button>
            <t t-foreach="items" t-as="item" t-key="item.id">
                <div t-att-class="{'active': item.selected}">
                    <t t-esc="item.name"/>
                </div>
            </t>
            <t t-slot="default"/>
        </div>
    </t>
</templates>
```

### Template Inheritance
```xml
<t t-name="my_module.MyTemplate" t-inherit="web.SomeTemplate" t-inherit-mode="extension">
    <xpath expr="//div[@class='target']" position="inside">
        <span>Added content</span>
    </xpath>
</t>
```

## SCSS Styling

### File Structure
```scss
// my_module/static/src/scss/my_styles.scss

// Import Bootstrap variables
@import "bootstrap/scss/functions";
@import "bootstrap/scss/variables";

// Custom variables
$my-primary: #007bff;
$my-spacing: 1rem;

// Component styles
.o_my_component {
    padding: $my-spacing;
    background: $my-primary;

    &__header {
        font-weight: bold;
    }

    &--active {
        border: 2px solid darken($my-primary, 10%);
    }
}

// Responsive
@include media-breakpoint-down(md) {
    .o_my_component {
        padding: $my-spacing / 2;
    }
}
```

## Debug vs Production Mode

### Debug Mode
```
URL: ?debug=1 or ?debug=assets

- Individual files served unminified
- Real-time file changes
- Full sourcemaps
- No caching
```

### Production Mode
```
- Single minified bundle per type
- SHA checksum versioning
- Browser caching enabled
- Optimized delivery
```

### Force Asset Regeneration
```python
# Clear asset cache
self.env['ir.qweb'].clear_caches()
```

## Registries

### Available Registries
```javascript
import { registry } from "@web/core/registry";

// Main registries
registry.category("services");      // Services
registry.category("fields");        // Field widgets
registry.category("views");         // View types
registry.category("actions");       // Client actions
registry.category("systray");       // Systray items
registry.category("user_menuitems"); // User menu items
registry.category("main_components"); // Main components
```

### Registering Components
```javascript
// Field widget
registry.category("fields").add("my_widget", {
    component: MyFieldComponent,
    supportedTypes: ["char", "text"],
});

// Systray item
registry.category("systray").add("my_systray", {
    Component: MySystrayComponent,
});

// Service
registry.category("services").add("myService", {
    dependencies: ["orm", "notification"],
    start(env, { orm, notification }) {
        return {
            doSomething() { /* ... */ }
        };
    },
});
```

## Asset Caching

### Checksum Versioning
```
Production URL: /web/assets/ab12cd3/web.assets_backend.min.js
                          ^^^^^^^
                          7-char SHA checksum
```

### Cache Invalidation
- Automatic on file modification
- Manual via Settings > Technical > Clear Assets

## Key Files

| File | Purpose |
|------|---------|
| `odoo/addons/base/models/assetsbundle.py` | Bundle processor |
| `odoo/addons/base/models/ir_asset.py` | Asset model |
| `addons/web/static/src/module_loader.js` | JS module loader |
| `addons/web/views/webclient_templates.xml` | Bundle definitions |

## Best Practices

1. **Use `@odoo-module`** annotation for ES6 modules
2. **Follow naming conventions** - `@module/path/component`
3. **Register in correct category** - fields, views, services
4. **Use SCSS variables** from Bootstrap
5. **Prefix CSS classes** with `o_` for Odoo components
6. **Test in production mode** before deployment
7. **Use glob patterns** for multiple files
8. **Prefer `after/before`** over manual ordering
