# Odoo 19.0 Deployment Guide

## Deployment Methods

| Method | Platform | Use Case |
|--------|----------|----------|
| Debian (.deb) | Ubuntu/Debian | Production |
| RPM | Fedora/RHEL | Production |
| pip | Any Python 3.10+ | Development |
| Docker | Any with Docker | Testing |
| Windows Installer | Windows 10/11 | Desktop |

## System Requirements

### Runtime
- **Python**: 3.10, 3.11, 3.12, or 3.13
- **PostgreSQL**: 12+ (13+ recommended)
- **OS**: Ubuntu 24.04 LTS, Debian 12, Fedora, RHEL, Windows 10/11

### Key Python Dependencies
- werkzeug (WSGI toolkit)
- gevent (async concurrency)
- psycopg2 (PostgreSQL adapter)
- lxml (XML processing)
- pillow (image processing)
- reportlab (PDF generation)

## Installation Methods

### Debian Package (Recommended for Production)

```bash
# Install package
sudo apt-get install ./odoo_19.0.0_all.deb

# Configure
sudo nano /etc/odoo/odoo.conf

# Start service
sudo systemctl enable odoo
sudo systemctl start odoo
```

**Directories Created**:
- Config: `/etc/odoo/`
- Data: `/var/lib/odoo/`
- Logs: `/var/log/odoo/`
- Binary: `/usr/bin/odoo`

### RPM Package

```bash
# Install
sudo dnf install ./odoo-19.0.0.rpm

# Start service
sudo systemctl enable odoo
sudo systemctl start odoo
```

### Python pip

```bash
pip install odoo

# Or from source
pip install ./odoo-19.0.0.tar.gz
```

### Development Setup

```bash
# Clone repository
git clone https://github.com/odoo/odoo.git
cd odoo

# Install dependencies
pip install -r requirements.txt

# Run
./odoo-bin -d mydb -i base
```

## Configuration

### Configuration File Location
- **Debian/RPM**: `/etc/odoo/odoo.conf`
- **Development**: `./odoo.conf` or `--config` flag

### Key Configuration Parameters

```ini
[options]
# Database
db_host = localhost
db_port = 5432
db_user = odoo
db_password = secretpassword
db_name = production

# HTTP Server
http_interface = 0.0.0.0
http_port = 8069

# Workers (0 = gevent mode)
workers = 4
limit_memory_soft = 2147483648
limit_memory_hard = 2684354560
limit_time_cpu = 60
limit_time_real = 120

# Logging
logfile = /var/log/odoo/odoo-server.log
loglevel = info

# Addons
addons_path = /usr/lib/python3/dist-packages/odoo/addons

# Admin
admin_passwd = admin_secret
```

### Environment Variables

| Variable | Maps To |
|----------|---------|
| `PGDATABASE` | db_name |
| `PGHOST` | db_host |
| `PGPORT` | db_port |
| `PGUSER` | db_user |
| `PGPASSWORD` | db_password |
| `ODOO_DEV` | dev_mode |

## WSGI Deployment (Gunicorn)

```bash
pip install gunicorn

gunicorn odoo.http:root \
  --pythonpath /path/to/odoo \
  --workers 4 \
  --bind 0.0.0.0:8069
```

## Docker Deployment

```bash
# Build and test
python3 setup/package.py --build-deb --test

# Or use official image
docker pull odoo:19.0
docker run -d -p 8069:8069 --name odoo odoo:19.0
```

## Service Management

```bash
# systemd commands
sudo systemctl start odoo
sudo systemctl stop odoo
sudo systemctl restart odoo
sudo systemctl status odoo
sudo systemctl enable odoo    # Auto-start
```

## Security Considerations

1. **Never run as root** - Odoo checks and refuses
2. **Don't use 'postgres' user** - Create dedicated odoo user
3. **Secure config file** - Permissions 0640
4. **Set admin password** - Required for database operations
5. **Use HTTPS** - Reverse proxy with nginx/Apache

## CI/CD Build System

**Build Command**: `python3 setup/package.py`

```bash
# Build all packages
python3 setup/package.py \
  --build-deb \
  --build-rpm \
  --build-tgz \
  --test \
  --sign
```

**Build Targets**:
- `--build-deb` - Debian package
- `--build-rpm` - RPM package
- `--build-tgz` - Source tarball
- `--build-win` - Windows installer
- `--build-iot` - IoT box installer

## Performance Tuning

### Worker Configuration

| Setting | Development | Production |
|---------|-------------|------------|
| workers | 0 (gevent) | 4-8 |
| limit_memory_soft | 2GB | 2GB |
| limit_memory_hard | 2.5GB | 2.5GB |
| limit_time_cpu | 60s | 60s |
| limit_time_real | 120s | 120s |

### Database

- Use PostgreSQL 13+ for best performance
- Configure connection pooling
- Set up read replicas for reporting

## Logging

- **Default**: stdout (captured by systemd)
- **File**: `--logfile /path/to/odoo.log`
- **Rotation**: Configured via logrotate (Debian package)

## Backup Strategy

```bash
# Database backup
pg_dump odoo_production > backup.sql

# Filestore backup
tar -czvf filestore.tar.gz /var/lib/odoo/filestore/
```

## Upgrade Process

1. Backup database and filestore
2. Stop Odoo service
3. Install new package/version
4. Run upgrade: `odoo-bin -u all -d database`
5. Start Odoo service
6. Verify functionality
