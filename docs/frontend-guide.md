# Odoo 19.0 Frontend Development Guide

## OWL Framework Overview

Odoo 19 uses OWL (Odoo Web Library) - a modern component-based JavaScript framework.

## Component Structure

### Basic Component

```javascript
/** @odoo-module */
import { Component, useState } from "@odoo/owl";

export class MyComponent extends Component {
    static template = "my_module.MyComponent";
    static props = {
        title: String,
        count: { type: Number, optional: true },
    };
    static defaultProps = {
        count: 0,
    };

    setup() {
        this.state = useState({ isOpen: false });
    }

    toggle() {
        this.state.isOpen = !this.state.isOpen;
    }
}
```

### QWeb Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="my_module.MyComponent">
        <div class="my-component">
            <h2 t-out="props.title"/>
            <button t-on-click="toggle">
                <t t-if="state.isOpen">Close</t>
                <t t-else="">Open</t>
            </button>
            <div t-if="state.isOpen" class="content">
                <t t-slot="default"/>
            </div>
        </div>
    </t>
</templates>
```

## QWeb Directives

| Directive | Purpose | Example |
|-----------|---------|---------|
| `t-if` | Conditional | `<t t-if="condition">` |
| `t-else` | Else branch | `<t t-else="">` |
| `t-foreach` | Loop | `<t t-foreach="items" t-as="item">` |
| `t-key` | Loop key | `t-key="item.id"` |
| `t-out` | Output text | `<span t-out="value"/>` |
| `t-esc` | Escaped output | `<span t-esc="value"/>` |
| `t-att-*` | Dynamic attribute | `t-att-class="className"` |
| `t-attf-*` | Formatted attribute | `t-attf-id="item-{{ id }}"` |
| `t-on-*` | Event handler | `t-on-click="handleClick"` |
| `t-ref` | DOM reference | `t-ref="myInput"` |
| `t-model` | Two-way binding | `t-model="state.value"` |
| `t-slot` | Named slot | `<t t-slot="header"/>` |

## Hooks

### Common Hooks

```javascript
import {
    useState,
    useRef,
    useEffect,
    onWillStart,
    onMounted,
    onWillUnmount
} from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";

export class MyComponent extends Component {
    setup() {
        // Reactive state
        this.state = useState({ count: 0 });

        // DOM reference
        this.inputRef = useRef("input");

        // Services
        this.orm = useService("orm");
        this.notification = useService("notification");

        // Lifecycle hooks
        onWillStart(async () => {
            // Before first render (async allowed)
            this.data = await this.loadData();
        });

        onMounted(() => {
            // After DOM is ready
            this.inputRef.el?.focus();
        });

        onWillUnmount(() => {
            // Cleanup before destroy
        });

        // Effect (runs when dependencies change)
        useEffect(() => {
            console.log("Count changed:", this.state.count);
        }, () => [this.state.count]);
    }
}
```

## Services

### Using Services

```javascript
import { useService } from "@web/core/utils/hooks";

export class MyComponent extends Component {
    setup() {
        this.orm = useService("orm");
        this.notification = useService("notification");
        this.dialog = useService("dialog");
        this.action = useService("action");
    }

    async saveRecord() {
        const id = await this.orm.create("my.model", [{ name: "Test" }]);
        this.notification.add("Record saved!", { type: "success" });
    }

    openDialog() {
        this.dialog.add(MyDialogComponent, {
            title: "Confirm",
            onConfirm: () => this.handleConfirm(),
        });
    }
}
```

### Available Services

| Service | Purpose |
|---------|---------|
| `orm` | Database operations |
| `notification` | Toast notifications |
| `dialog` | Modal dialogs |
| `action` | Execute actions |
| `rpc` | Raw RPC calls |
| `user` | Current user info |
| `company` | Company info |
| `hotkey` | Keyboard shortcuts |

### Creating a Service

```javascript
import { registry } from "@web/core/registry";

const myService = {
    dependencies: ["notification"],

    start(env, { notification }) {
        return {
            doSomething() {
                notification.add("Something done!");
            }
        };
    }
};

registry.category("services").add("my_service", myService);
```

## Registry System

### Main Categories

```javascript
import { registry } from "@web/core/registry";

// Register a field type
registry.category("fields").add("my_field", myFieldDefinition);

// Register a view type
registry.category("views").add("my_view", myViewDefinition);

// Register a service
registry.category("services").add("my_service", myService);

// Register a main component
registry.category("main_components").add("MyComponent", {
    Component: MyComponent,
});
```

## Custom Field Widget

```javascript
import { Component } from "@odoo/owl";
import { registry } from "@web/core/registry";
import { standardFieldProps } from "@web/views/fields/standard_field_props";

export class MyField extends Component {
    static template = "my_module.MyField";
    static props = { ...standardFieldProps };

    get value() {
        return this.props.record.data[this.props.name];
    }

    onChange(ev) {
        this.props.record.update({
            [this.props.name]: ev.target.value
        });
    }
}

export const myField = {
    component: MyField,
    supportedTypes: ["char"],
};

registry.category("fields").add("my_field", myField);
```

```xml
<t t-name="my_module.MyField">
    <input
        type="text"
        t-att-value="value"
        t-on-change="onChange"
        t-att-readonly="props.readonly"
    />
</t>
```

## Asset Bundling

### Manifest Configuration

```python
# __manifest__.py
{
    'name': 'My Module',
    'assets': {
        'web.assets_backend': [
            # JavaScript
            'my_module/static/src/js/**/*.js',
            # XML templates
            'my_module/static/src/xml/**/*.xml',
            # SCSS styles
            'my_module/static/src/scss/**/*.scss',
        ],
        'web.assets_frontend': [
            'my_module/static/src/public/**/*',
        ],
    }
}
```

### Asset Operations

```python
'assets': {
    'web.assets_backend': [
        # Include bundle
        ('include', 'web._assets_helpers'),

        # Add files
        'my_module/static/src/**/*',

        # Remove file
        ('remove', 'other_module/static/src/unwanted.js'),

        # Reorder
        ('after', 'web/static/src/core/utils.js',
                  'my_module/static/src/patch.js'),
    ]
}
```

## File Organization

```
my_module/
├── static/
│   └── src/
│       ├── js/
│       │   ├── components/
│       │   │   ├── my_component.js
│       │   │   └── my_component.xml
│       │   └── services/
│       │       └── my_service.js
│       ├── views/
│       │   └── my_view/
│       │       ├── my_view.js
│       │       └── my_view.xml
│       └── scss/
│           └── my_styles.scss
└── __manifest__.py
```

## Common Patterns

### Form Field Integration

```javascript
// Use in form view XML
// <field name="my_field" widget="my_widget"/>
```

### Action Handler

```javascript
import { registry } from "@web/core/registry";

function myActionHandler(env, action) {
    // Handle the action
    console.log("Action:", action);
}

registry.category("actions").add("my_action_tag", myActionHandler);
```

### Patching Existing Components

```javascript
import { patch } from "@web/core/utils/patch";
import { FormController } from "@web/views/form/form_controller";

patch(FormController.prototype, {
    setup() {
        super.setup();
        // Additional setup
    },

    async save() {
        // Custom save logic
        return super.save();
    }
});
```

## Debugging

```javascript
// Access OWL dev tools
owl.dev = true;

// Log component tree
console.log(this.__owl__);

// Access environment
console.log(this.env);
```

## Best Practices

1. **Use hooks** for lifecycle and side effects
2. **Keep components small** and focused
3. **Use services** for shared logic
4. **Avoid direct DOM manipulation** - use refs
5. **Use `useState`** for reactive state
6. **Name templates** with module prefix
7. **Validate props** with static props definition
