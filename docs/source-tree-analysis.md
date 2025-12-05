# Odoo 19.0 Source Tree Analysis

## Project Root Structure

```
odoo/                           # Project Root
├── odoo-bin                    # ENTRY POINT - Main executable script
├── setup.py                    # Python package setup
├── setup.cfg                   # Setup configuration
├── requirements.txt            # Python dependencies
├── ruff.toml                   # Linter configuration
│
├── README.md                   # Project documentation
├── CONTRIBUTING.md             # Contribution guidelines
├── SECURITY.md                 # Security policy
├── LICENSE                     # LGPL-3 license
├── COPYRIGHT                   # Copyright information
├── MANIFEST.in                 # Package manifest
│
├── odoo/                       # CORE FRAMEWORK
│   ├── __main__.py             # Module entry point
│   ├── release.py              # Version information (19.0.0)
│   ├── http.py                 # HTTP/WSGI layer
│   ├── sql_db.py               # Database connection pool
│   ├── netsvc.py               # Network services
│   ├── exceptions.py           # Custom exceptions
│   ├── loglevels.py            # Logging configuration
│   ├── init.py                 # Initialization routines
│   ├── import_xml.rng          # XML validation schema
│   │
│   ├── orm/                    # OBJECT-RELATIONAL MAPPER
│   │   ├── models.py           # Base model classes
│   │   ├── fields.py           # Field definitions
│   │   ├── fields_*.py         # Specialized field types
│   │   ├── environments.py     # Environment management
│   │   ├── registry.py         # Model registry
│   │   ├── domains.py          # Domain expression parser
│   │   ├── commands.py         # ORM commands
│   │   ├── decorators.py       # API decorators
│   │   └── utils.py            # ORM utilities
│   │
│   ├── api/                    # API LAYER
│   │   └── ...                 # RPC and API handling
│   │
│   ├── cli/                    # COMMAND LINE INTERFACE
│   │   ├── command.py          # Base command class
│   │   ├── server.py           # Start server command
│   │   ├── shell.py            # Interactive shell
│   │   ├── scaffold.py         # Module scaffolding
│   │   ├── db.py               # Database management
│   │   ├── deploy.py           # Module deployment
│   │   ├── i18n.py             # Translation tools
│   │   ├── cloc.py             # Lines of code counter
│   │   ├── populate.py         # Demo data generation
│   │   ├── neutralize.py       # Database neutralization
│   │   ├── obfuscate.py        # Data obfuscation
│   │   ├── upgrade_code.py     # Code upgrade tools
│   │   └── templates/          # Scaffold templates
│   │
│   ├── fields/                 # FIELD TYPE SYSTEM
│   │   └── ...                 # Field implementations
│   │
│   ├── models/                 # MODEL UTILITIES
│   │   └── ...                 # Model helpers
│   │
│   ├── modules/                # MODULE LOADING
│   │   └── ...                 # Module management
│   │
│   ├── service/                # SERVICES
│   │   └── ...                 # Background services
│   │
│   ├── tools/                  # UTILITY FUNCTIONS
│   │   └── ...                 # Various helpers
│   │
│   ├── tests/                  # CORE TESTS
│   │   └── ...                 # Framework tests
│   │
│   ├── upgrade/                # UPGRADE SCRIPTS
│   │   └── ...                 # Database migrations
│   │
│   ├── upgrade_code/           # CODE UPGRADES
│   │   └── ...                 # Code transformation
│   │
│   ├── osv/                    # OSV COMPATIBILITY
│   │   └── ...                 # Legacy compatibility
│   │
│   ├── _monkeypatches/         # RUNTIME PATCHES
│   │   └── ...                 # Compatibility patches
│   │
│   └── addons/                 # CORE ADDONS
│       └── base/               # BASE MODULE (Required)
│           ├── __manifest__.py # Module manifest
│           ├── models/         # Core models (res.*, ir.*)
│           ├── views/          # View definitions
│           ├── data/           # Initial data
│           ├── security/       # Access control
│           └── tests/          # Module tests
│
├── addons/                     # BUSINESS MODULES (600 addons)
│   │
│   │── ACCOUNTING DOMAIN
│   ├── account/                # Invoicing & Accounting
│   ├── account_payment/        # Payment processing
│   ├── account_check_printing/ # Check printing
│   ├── account_edi/            # Electronic invoicing
│   ├── account_peppol/         # PEPPOL integration
│   │
│   │── SALES DOMAIN
│   ├── sale/                   # Sales orders
│   ├── sale_management/        # Sales management
│   ├── sale_loyalty/           # Loyalty programs
│   ├── sale_subscription/      # Subscriptions
│   │
│   │── PURCHASE DOMAIN
│   ├── purchase/               # Purchase orders
│   ├── purchase_requisition/   # Purchase requisitions
│   ├── purchase_stock/         # Purchase-Stock integration
│   │
│   │── CRM DOMAIN
│   ├── crm/                    # Leads & Opportunities
│   ├── crm_livechat/           # Live chat integration
│   ├── crm_sms/                # SMS integration
│   │
│   │── HR DOMAIN
│   ├── hr/                     # Employee management
│   ├── hr_attendance/          # Attendance tracking
│   ├── hr_expense/             # Expense management
│   ├── hr_holidays/            # Leave management
│   ├── hr_recruitment/         # Recruitment
│   │
│   │── INVENTORY DOMAIN
│   ├── stock/                  # Warehouse management
│   ├── stock_account/          # Stock accounting
│   ├── stock_picking_batch/    # Batch picking
│   │
│   │── MANUFACTURING DOMAIN
│   ├── mrp/                    # Manufacturing
│   ├── mrp_account/            # MRP accounting
│   ├── mrp_subcontracting/     # Subcontracting
│   │
│   │── PROJECT DOMAIN
│   ├── project/                # Project management
│   ├── project_todo/           # To-do lists
│   │
│   │── WEBSITE DOMAIN
│   ├── website/                # Website builder
│   ├── website_sale/           # E-commerce
│   ├── website_blog/           # Blog
│   ├── website_forum/          # Forums
│   ├── website_event/          # Event management
│   │
│   │── COMMUNICATION DOMAIN
│   ├── mail/                   # Messaging & Email
│   ├── sms/                    # SMS messaging
│   ├── im_livechat/            # Live chat
│   ├── discuss/                # Discuss app
│   │
│   │── POINT OF SALE
│   ├── point_of_sale/          # POS application
│   ├── pos_sale/               # POS-Sales integration
│   │
│   │── LOCALIZATION (l10n_*)
│   ├── l10n_generic_coa/       # Generic chart of accounts
│   ├── l10n_us/                # US localization
│   ├── l10n_in/                # India localization
│   ├── l10n_*/                 # Country-specific (100+ modules)
│   │
│   │── PAYMENT PROVIDERS
│   ├── payment/                # Payment framework
│   ├── payment_stripe/         # Stripe
│   ├── payment_paypal/         # PayPal
│   ├── payment_razorpay/       # Razorpay
│   ├── payment_*/              # Other providers
│   │
│   │── AUTHENTICATION
│   ├── auth_signup/            # User signup
│   ├── auth_ldap/              # LDAP authentication
│   ├── auth_oauth/             # OAuth providers
│   ├── auth_totp/              # 2FA TOTP
│   ├── auth_passkey/           # Passkey authentication
│   │
│   │── UTILITIES
│   ├── base_automation/        # Server actions automation
│   ├── base_import/            # Data import
│   ├── base_geolocalize/       # Geolocation
│   ├── bus/                    # Longpolling/WebSocket
│   ├── calendar/               # Calendar & events
│   └── ...                     # 600 total addons
│
├── setup/                      # INSTALLATION & PACKAGING
│   ├── package.py              # Build automation
│   ├── odoo                    # Entry script
│   ├── odoo-wsgi.example.py    # WSGI configuration
│   ├── debinstall.sh           # Debian dependency installer
│   ├── rpm/                    # RPM packaging
│   │   └── odoo.spec           # RPM spec file
│   ├── win32/                  # Windows installer
│   │   ├── setup.nsi           # NSIS installer script
│   │   └── setup-iot.nsi       # IoT box installer
│   └── package.df*             # Docker build files
│
├── debian/                     # DEBIAN PACKAGING
│   ├── control                 # Package metadata
│   ├── rules                   # Build rules
│   ├── postinst                # Post-install script
│   ├── postrm                  # Post-removal script
│   ├── odoo.service            # systemd service
│   ├── odoo.conf               # Default configuration
│   └── logrotate               # Log rotation config
│
├── doc/                        # DOCUMENTATION
│   └── cla/                    # Contributor License Agreements
│       ├── ccla-1.0.md         # Corporate CLA template
│       ├── corporate/          # Signed corporate CLAs
│       └── individual/         # Signed individual CLAs
│
├── docs/                       # GENERATED DOCUMENTATION
│   └── ...                     # AI-generated docs (this workflow)
│
└── .github/                    # GITHUB CONFIGURATION
    ├── PULL_REQUEST_TEMPLATE.md
    └── workflows/              # CI/CD workflows (if any)
```

## Critical Directories Explained

### `/odoo/` - Core Framework

The heart of Odoo, containing:
- **ORM System**: Database abstraction layer in `orm/`
- **HTTP Layer**: WSGI application in `http.py`
- **CLI Tools**: Command-line interface in `cli/`
- **Base Module**: Required `base` addon in `addons/base/`

### `/addons/` - Business Modules

600 independent addon modules organized by business domain:
- Each addon is self-contained with models, views, controllers
- Addons declare dependencies via `__manifest__.py`
- Prefixed by domain (account_, sale_, hr_, etc.)

### `/setup/` - Packaging & Distribution

Build and packaging tools:
- Multi-platform builds (Debian, RPM, Windows, Docker)
- WSGI deployment configuration
- Dependency management

## Addon Module Structure

Every addon follows this standard structure:

```
addons/example_module/
├── __init__.py                 # Module initialization
├── __manifest__.py             # Module metadata & dependencies
│
├── models/                     # ORM model definitions
│   ├── __init__.py
│   └── example_model.py
│
├── views/                      # XML view definitions
│   └── example_views.xml       # Form, tree, kanban views
│
├── controllers/                # HTTP controllers
│   ├── __init__.py
│   └── main.py                 # Route definitions
│
├── wizard/                     # Transient models (wizards)
│   ├── __init__.py
│   └── example_wizard.py
│
├── report/                     # Report definitions
│   ├── __init__.py
│   └── example_report.py
│
├── security/                   # Access control
│   ├── ir.model.access.csv     # Model access rights
│   └── example_security.xml    # Security groups & rules
│
├── data/                       # Static/demo data
│   ├── example_data.xml        # Initial data
│   └── example_demo.xml        # Demo data
│
├── static/                     # Static assets
│   ├── src/
│   │   ├── js/                 # JavaScript/OWL components
│   │   ├── css/                # Stylesheets
│   │   ├── scss/               # SCSS files
│   │   └── xml/                # QWeb templates
│   ├── description/
│   │   └── icon.png            # Module icon
│   └── lib/                    # Third-party libraries
│
├── tests/                      # Unit & integration tests
│   ├── __init__.py
│   └── test_example.py
│
└── i18n/                       # Translations
    └── example.pot             # Translation template
```

## Entry Points

| Entry Point | Location | Purpose |
|-------------|----------|---------|
| Main CLI | `odoo-bin` | Start server, shell, commands |
| Module Entry | `odoo/__main__.py` | Python -m odoo |
| WSGI App | `odoo.http:root` | Gunicorn/uWSGI |
| Base Module | `odoo/addons/base/` | Core functionality |

## Key Files Reference

| File | Purpose |
|------|---------|
| `odoo/release.py` | Version info (19.0.0) |
| `odoo/http.py` | HTTP request handling |
| `odoo/sql_db.py` | Database connection pool |
| `odoo/orm/models.py` | ORM base classes |
| `odoo/orm/fields.py` | Field type definitions |
| `odoo/cli/server.py` | Server startup |
| `requirements.txt` | Python dependencies |
| `setup.py` | Package installation |

## Module Count by Domain

| Domain | Count | Examples |
|--------|-------|----------|
| Accounting | ~20 | account, account_payment |
| Sales | ~15 | sale, sale_management |
| Purchase | ~10 | purchase, purchase_stock |
| HR | ~20 | hr, hr_holidays, hr_expense |
| Stock | ~15 | stock, stock_account |
| Website | ~30 | website, website_sale |
| Localization | ~100 | l10n_* modules |
| Payment | ~20 | payment_* providers |
| **Total** | **600** | |
