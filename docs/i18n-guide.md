# Odoo 19.0 Internationalization Guide

## Translation System Overview

Odoo uses GNU gettext-compatible `.po` files with custom extensions for model/field translations.

## Translation File Formats

### PO/POT Files
- Location: `<module>/i18n/` or `<module>/i18n_extra/`
- Template: `<module>.pot`
- Translations: `<lang_code>.po` (e.g., `fr.po`, `de.po`)

### File Structure
```
#. module: module_name
#: model:ir.model.fields,field_description:module.field_id
msgid "Field Label"
msgstr "Translated Label"

#: code:addons/module/models/model.py:123
msgid "Error message"
msgstr "Message d'erreur"
```

## Translation APIs

### Basic Translation Function
```python
from odoo import _

# Simple translation
message = _("Hello World")

# With formatting (preferred)
message = _("Hello %s", name)
message = _("Count: %(count)d items", count=total)
```

### Lazy Translation (Module-Level)
```python
from odoo.tools.translate import LazyTranslate

_lt = LazyTranslate(__name__)

# Define at module level
MESSAGES = {
    'success': _lt("Operation completed"),
    'error': _lt("An error occurred"),
}

# Use in context with environment
def some_method(self):
    return self.env._(MESSAGES['success'])
```

## Field Translation

### Translatable Fields
```python
class MyModel(models.Model):
    _name = 'my.model'

    # Simple translated text
    name = fields.Char(translate=True)

    # HTML with translation
    description = fields.Html(translate=True)

    # Custom translate function
    content = fields.Html(translate=html_translate)
```

### Translation Functions
| Function | Purpose |
|----------|---------|
| `translate=True` | Standard text translation |
| `html_translate` | HTML content with preserved tags |
| `xml_translate` | XML content translation |

## Language Management

### res.lang Model Fields
```python
code = 'en_US'          # Locale code
iso_code = 'en'         # ISO code for .po selection
url_code = 'en'         # URL-friendly code
direction = 'ltr'       # 'ltr' or 'rtl'
date_format = '%m/%d/%Y'
time_format = '%H:%M:%S'
decimal_point = '.'
thousands_sep = ','
grouping = '[3,0]'      # Number grouping pattern
```

### Language Installation
```python
# Via wizard
self.env['base.language.install'].create({
    'lang_ids': [(4, lang.id)]
}).lang_install()

# Via code
from odoo.tools.translate import load_language
load_language(cr, 'fr_FR')
```

## Number & Date Formatting

### Number Formatting
```python
# Format numbers per locale
lang = self.env['res.lang']._get_data(code='de_DE')
formatted = lang.format('%d', 1234567, grouping=True)
# Result: "1.234.567" (German grouping)
```

### Grouping Patterns
| Pattern | Description | Example |
|---------|-------------|---------|
| `[3,0]` | Standard (thousands) | 1,234,567 |
| `[3,2,0]` | Indian numbering | 12,34,567 |

## RTL Support

### Direction Detection
```python
lang_data = self.env['res.lang']._get_data(code=self.env.lang)
if lang_data.direction == 'rtl':
    # Apply RTL-specific logic
    pass
```

### RTL Languages
Arabic, Hebrew, Farsi, Urdu - automatically detected via `direction` field.

## Translation Import/Export

### Export Translations
```bash
./odoo-bin --modules=my_module --i18n-export=my_module.pot
```

### Import Translations
```python
# Via TranslationImporter
from odoo.tools.translate import TranslationImporter

importer = TranslationImporter(cr, overwrite=True)
importer.load_file('/path/to/fr.po', 'fr_FR')
importer.save()
```

## Context-Based Language

### Switch Language in Code
```python
# Use specific language
record_fr = record.with_context(lang='fr_FR')
translated_name = record_fr.name

# User's language
user_lang = self.env.user.lang
```

## Key Files

| File | Purpose |
|------|---------|
| `odoo/tools/translate.py` | Core translation engine |
| `odoo/tools/i18n.py` | Locale utilities |
| `odoo/addons/base/models/res_lang.py` | Language configuration |
| `odoo/addons/base/wizard/base_language_install.py` | Language installation |

## Best Practices

1. **Use `_()` for all user-facing strings**
2. **Use format arguments** - avoid string concatenation
3. **Mark fields with `translate=True`** for translatable content
4. **Use `_lt()` for module-level strings** that need lazy evaluation
5. **Include translator comments** in PO files for context
6. **Test RTL layouts** with Arabic or Hebrew
7. **Run translation extraction** before releases
