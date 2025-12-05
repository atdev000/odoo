# Odoo 19.0 Documentation Index

## Project Overview

- **Name**: Odoo ERP
- **Version**: 19.0.0 (Final)
- **Type**: Monolith with modular addon architecture
- **Language**: Python 3.10-3.13
- **Database**: PostgreSQL 13+
- **License**: LGPL-3

## Quick Reference

| Metric | Value |
|--------|-------|
| **Addons** | 600 modules |
| **Models** | 1,259+ |
| **API Endpoints** | 211+ |
| **Architecture** | Layered MVC with Plugin System |
| **Entry Point** | `odoo-bin` |

## Generated Documentation

### Core Documentation

- [Project Overview](./project-overview.md) - High-level project summary
- [Architecture](./architecture.md) - System architecture and design patterns
- [Source Tree Analysis](./source-tree-analysis.md) - Code structure and organization

### Technical Documentation

- [Data Models](./data-models.md) - ORM models, fields, and relationships
- [API Contracts](./api-contracts.md) - HTTP endpoints, routes, and controllers

### Guides

#### Development
- [Development Guide](./development-guide.md) - Setup, coding, and testing
- [Testing Guide](./testing-guide.md) - Test patterns, assertions, and coverage
- [Frontend Guide](./frontend-guide.md) - OWL components, QWeb templates, and services
- [Assets Guide](./assets-guide.md) - JavaScript modules, SCSS, and bundling
- [Views Guide](./views-guide.md) - Form, list, kanban views and inheritance

#### Operations
- [Deployment Guide](./deployment-guide.md) - Installation and configuration
- [Migration Guide](./migration-guide.md) - Version upgrades and data migration
- [Performance Guide](./performance-guide.md) - Caching, profiling, and optimization

#### System Features
- [Security Guide](./security-guide.md) - Authentication, access control, and best practices
- [Integration Guide](./integration-guide.md) - APIs, webhooks, and external connectivity
- [Automation Guide](./automation-guide.md) - Cron jobs, server actions, and workflows
- [Reporting Guide](./reporting-guide.md) - QWeb reports, PDF generation, and exports
- [i18n Guide](./i18n-guide.md) - Internationalization and localization
- [Mail Guide](./mail-guide.md) - Messaging, email templates, and activities
- [Multi-Company Guide](./multicompany-guide.md) - Company rules and data isolation
- [Domain Guide](./domain-guide.md) - Search syntax and operators
- [Attachments Guide](./attachments-guide.md) - Binary fields and document storage

## Existing Documentation

### Root Level
- [README.md](../README.md) - Main project overview
- [CONTRIBUTING.md](../CONTRIBUTING.md) - Contribution guidelines
- [SECURITY.md](../SECURITY.md) - Security policy
- [LICENSE](../LICENSE) - LGPL-3 license

### External
- [Official Docs](https://www.odoo.com/documentation/19.0/) - Odoo official documentation
- [GitHub Wiki](https://github.com/odoo/odoo/wiki) - Community wiki

## Getting Started

### Prerequisites
```bash
# Python 3.10+ and PostgreSQL 13+
python3 --version
psql --version
```

### Quick Install (Development)
```bash
git clone https://github.com/odoo/odoo.git
cd odoo
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
./odoo-bin -d mydb -i base
```

### Access
- Web Interface: http://localhost:8069
- Backend: http://localhost:8069/web
- Default credentials: admin / admin

## Module Categories

| Domain | Key Modules | Purpose |
|--------|-------------|---------|
| **Accounting** | account, account_payment | Financial management |
| **Sales** | sale, sale_management | Order processing |
| **Purchase** | purchase, purchase_stock | Vendor management |
| **Inventory** | stock, stock_account | Warehouse operations |
| **Manufacturing** | mrp, mrp_account | Production planning |
| **HR** | hr, hr_holidays, hr_expense | Employee management |
| **CRM** | crm, crm_livechat | Customer relationships |
| **Project** | project, project_todo | Task management |
| **Website** | website, website_sale | E-commerce & CMS |
| **POS** | point_of_sale, pos_sale | Retail operations |

## Architecture Summary

```
┌──────────────────────────────────────┐
│       Presentation (OWL + QWeb)      │
├──────────────────────────────────────┤
│    Controllers (Werkzeug + HTTP)     │
├──────────────────────────────────────┤
│    Business Logic (600 Addons)       │
├──────────────────────────────────────┤
│    ORM Layer (Custom Odoo ORM)       │
├──────────────────────────────────────┤
│    Database (PostgreSQL)             │
└──────────────────────────────────────┘
```

## Key Paths

| Path | Purpose |
|------|---------|
| `odoo/` | Core framework |
| `odoo/orm/` | Object-Relational Mapper |
| `odoo/cli/` | Command-line interface |
| `odoo/http.py` | HTTP/WSGI layer |
| `odoo/addons/base/` | Base module (required) |
| `addons/` | Business modules (600) |
| `setup/` | Packaging and deployment |

## Support

- **Issues**: https://github.com/odoo/odoo/issues
- **Forum**: https://www.odoo.com/forum/help-1
- **Security**: https://www.odoo.com/security-report

---

*Generated: 2025-12-05*
*Scan Level: Exhaustive*
*Workflow: document-project v1.2.0*
