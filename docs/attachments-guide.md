# Odoo 19.0 Attachments & Documents Guide

## ir.attachment Model

### Key Fields
| Field | Type | Description |
|-------|------|-------------|
| `name` | Char | Filename |
| `type` | Selection | `'url'` or `'binary'` |
| `url` | Char | URL for url-type attachments |
| `datas` | Binary | Base64-encoded content |
| `raw` | Binary | Raw binary content |
| `res_model` | Char | Linked model name |
| `res_id` | Integer | Linked record ID |
| `res_field` | Char | Linked binary field |
| `mimetype` | Char | MIME type (auto-detected) |
| `file_size` | Integer | Size in bytes |
| `checksum` | Char | SHA1 hash |
| `public` | Boolean | Publicly accessible |
| `access_token` | Char | Token for external access |

## Creating Attachments

### Basic Attachment
```python
attachment = self.env['ir.attachment'].create({
    'name': 'document.pdf',
    'type': 'binary',
    'datas': base64.b64encode(file_content),
    'res_model': 'sale.order',
    'res_id': order.id,
})
```

### URL Attachment
```python
attachment = self.env['ir.attachment'].create({
    'name': 'External Document',
    'type': 'url',
    'url': 'https://example.com/document.pdf',
})
```

### Attachment to Record
```python
# Via message_post
record.message_post(
    body="See attached document",
    attachment_ids=[attachment.id]
)
```

## Binary Fields

### Standard Binary Field
```python
class MyModel(models.Model):
    _name = 'my.model'

    # Stored in ir.attachment (default)
    document = fields.Binary(string='Document', attachment=True)
    document_name = fields.Char()

    # Stored in database column
    small_data = fields.Binary(attachment=False)
```

### Image Field
```python
class MyModel(models.Model):
    _name = 'my.model'

    # Auto-resized image
    image = fields.Image(
        max_width=1920,
        max_height=1920,
    )

    # Smaller versions (computed)
    image_128 = fields.Image(
        related='image',
        max_width=128,
        max_height=128,
        store=True
    )
```

## Storage Configuration

### Storage Locations
```ini
# odoo.conf

# Filesystem storage (default)
data_dir = /var/lib/odoo

# Force database storage
# ir_attachment.location = db
```

### Storage Types
| Type | Config | Location |
|------|--------|----------|
| Filesystem | `file` | `{data_dir}/filestore/{db}/` |
| Database | `db` | `ir_attachment.db_datas` column |
| Cloud | module | Azure/Google Cloud Storage |

### File Path Structure
```
{data_dir}/filestore/{database}/
├── ab/
│   └── ab123456789...  # SHA1 hash
├── cd/
│   └── cd987654321...
└── ...
```

## Access Control

### Public Attachments
```python
# Create public attachment
attachment = self.env['ir.attachment'].create({
    'name': 'public_image.png',
    'datas': image_data,
    'public': True,
})

# Access URL
url = f'/web/image/{attachment.id}'
```

### Access Token
```python
# Generate access token
attachment.generate_access_token()

# Access with token
url = f'/web/content/{attachment.id}?access_token={attachment.access_token}'
```

### Access Rules
```python
# Attachment access follows linked record access
# If user can read sale.order #5, they can read its attachments

# Orphan attachments (no res_model/res_id)
# - Creator can access
# - System users can access
```

## Serving Attachments

### HTTP Routes
```python
# Image (with resize)
/web/image/{model}/{id}/{field}
/web/image/{model}/{id}/{field}/{width}x{height}

# Content download
/web/content/{id}
/web/content/{id}/{filename}

# With access token
/web/content/{id}?access_token={token}
```

### In Views
```xml
<!-- Image widget -->
<field name="image" widget="image"/>

<!-- Binary download -->
<field name="document" filename="document_name"/>

<!-- Image URL in template -->
<img t-attf-src="/web/image/#{record._name}/#{record.id}/image_128"/>
```

## Image Processing

### Automatic Processing
- EXIF orientation correction
- Resolution verification (max 50Mpx)
- Auto-resize to max dimensions
- Quality optimization

### Configuration
```python
# System parameters
base.image_autoresize_max_px = '1920x1920'
base.image_autoresize_quality = 80
```

### Manual Processing
```python
from odoo.tools.image import image_process

processed = image_process(
    image_data,
    size=(800, 600),
    crop='center',
    quality=85
)
```

## Document Indexing

### Automatic Indexing
```python
# Text files automatically indexed
# Stored in index_content field
# Searchable via domain
[('index_content', 'ilike', 'search term')]
```

### Supported Formats
- Plain text files (text/*)
- PDF (with attachment_indexation module)
- Office documents (with additional modules)

## Garbage Collection

### Automatic Cleanup
```python
# Runs via autovacuum
# Removes orphan files from filestore
# Safe for deduplicated files

# Triggered on attachment delete/update
# Uses checklist directory pattern
```

### Manual Cleanup
```python
# Force garbage collection
self.env['ir.attachment']._gc_file_store()
```

## Key Files

| File | Purpose |
|------|---------|
| `odoo/addons/base/models/ir_attachment.py` | Attachment model |
| `odoo/tools/image.py` | Image processing |
| `odoo/orm/fields_binary.py` | Binary/Image fields |
| `addons/web/controllers/binary.py` | HTTP serving |

## Common Patterns

### Upload Handler
```python
@http.route('/my/upload', type='http', auth='user', methods=['POST'])
def upload_file(self, file, **kwargs):
    attachment = request.env['ir.attachment'].create({
        'name': file.filename,
        'datas': base64.b64encode(file.read()),
        'res_model': 'my.model',
        'res_id': kwargs.get('record_id'),
    })
    return json.dumps({'id': attachment.id})
```

### Download Link
```python
def get_download_url(self):
    return f'/web/content/{self.attachment_id.id}?download=true'
```

### Image Resize on Read
```xml
<img t-att-src="'/web/image/%s/%s/image/128x128' % (record._name, record.id)"/>
```

## Best Practices

1. **Use `attachment=True`** for large binary fields
2. **Set `max_width/max_height`** on Image fields
3. **Use `public=True`** only when needed
4. **Generate access tokens** for external sharing
5. **Link attachments to records** via res_model/res_id
6. **Use image_128 variants** for thumbnails
7. **Configure filestore** on shared storage for multi-server
