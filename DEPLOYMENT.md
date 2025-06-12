# Deployment Guide

## Overview

This guide covers deployment procedures for Web Value Task Flow in various environments.

## Table of Contents

1. [Requirements](#requirements)
2. [Environment Setup](#environment-setup)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Database Setup](#database-setup)
6. [Web Server Configuration](#web-server-configuration)
7. [Security Measures](#security-measures)
8. [Monitoring](#monitoring)
9. [Backup Procedures](#backup-procedures)
10. [Maintenance](#maintenance)

## Requirements

### System Requirements

- PHP 8.1+
- MySQL 5.7+
- Apache 2.4+ or Nginx 1.18+
- Composer 2.0+
- Node.js 16+
- SSL Certificate

### Server Specifications

Minimum requirements:
- CPU: 2 cores
- RAM: 4GB
- Storage: 20GB SSD
- Bandwidth: 100GB/month

Recommended:
- CPU: 4 cores
- RAM: 8GB
- Storage: 50GB SSD
- Bandwidth: 500GB/month

## Environment Setup

### Directory Structure

```
/var/www/task-flow/
├── public/          # Web root
├── src/            # Application source
├── config/         # Configuration files
├── storage/        # File uploads, cache
├── logs/          # Application logs
└── backup/        # Backup directory
```

### File Permissions

```bash
# Set ownership
chown -R www-data:www-data /var/www/task-flow

# Set directory permissions
find /var/www/task-flow -type d -exec chmod 755 {} \;

# Set file permissions
find /var/www/task-flow -type f -exec chmod 644 {} \;

# Set storage permissions
chmod -R 775 /var/www/task-flow/storage
chmod -R 775 /var/www/task-flow/logs
```

## Installation

### Fresh Installation

```bash
# Clone repository
git clone https://github.com/webvalue/task-flow.git /var/www/task-flow

# Install dependencies
cd /var/www/task-flow
composer install --no-dev --optimize-autoloader
npm install --production

# Build assets
npm run build

# Set up configuration
cp config.example.php config.php
vim config.php  # Edit configuration

# Initialize application
php init.php
```

### Updating Existing Installation

```bash
# Pull latest changes
git pull origin main

# Install dependencies
composer install --no-dev --optimize-autoloader
npm install --production

# Build assets
npm run build

# Clear cache
php clear-cache.php

# Run database migrations
php migrate.php
```

## Configuration

### Environment Configuration

Edit config.php:
```php
// Database
define('DB_HOST', 'localhost');
define('DB_NAME', 'task_flow');
define('DB_USER', 'production_user');
define('DB_PASS', 'secure_password');

// Application
define('APP_URL', 'https://tasks.webvalue.com');
define('APP_ENV', 'production');
define('APP_DEBUG', false);

// Security
define('SECURE_SESSION', true);
define('CSRF_PROTECTION', true);
```

### PHP Configuration

Edit php.ini:
```ini
; Memory
memory_limit = 256M
max_execution_time = 60

; Upload limits
upload_max_filesize = 10M
post_max_size = 10M

; Error handling
display_errors = Off
log_errors = On
error_log = /var/www/task-flow/logs/php_errors.log
```

## Database Setup

### Creating Database

```sql
CREATE DATABASE task_flow CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'task_flow'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON task_flow.* TO 'task_flow'@'localhost';
FLUSH PRIVILEGES;
```

### MySQL Configuration

Edit my.cnf:
```ini
[mysqld]
innodb_buffer_pool_size = 1G
max_connections = 150
query_cache_size = 32M
```

## Web Server Configuration

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName tasks.webvalue.com
    DocumentRoot /var/www/task-flow/public
    
    <Directory /var/www/task-flow/public>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/task-flow-error.log
    CustomLog ${APACHE_LOG_DIR}/task-flow-access.log combined
    
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>
```

### Nginx Configuration

```nginx
server {
    listen 80;
    server_name tasks.webvalue.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name tasks.webvalue.com;
    root /var/www/task-flow/public;
    
    ssl_certificate /etc/letsencrypt/live/tasks.webvalue.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tasks.webvalue.com/privkey.pem;
    
    index index.php;
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    
    location ~ /\.ht {
        deny all;
    }
}
```

## Security Measures

### SSL Configuration

```bash
# Install Let's Encrypt
apt-get install certbot python3-certbot-apache

# Generate certificate
certbot --apache -d tasks.webvalue.com
```

### Firewall Setup

```bash
# Allow SSH, HTTP, HTTPS
ufw allow 22
ufw allow 80
ufw allow 443
ufw enable
```

### Security Headers

```apache
# Add security headers
Header set X-Frame-Options "SAMEORIGIN"
Header set X-XSS-Protection "1; mode=block"
Header set X-Content-Type-Options "nosniff"
Header set Referrer-Policy "strict-origin-when-cross-origin"
Header set Content-Security-Policy "default-src 'self'"
```

## Monitoring

### Log Monitoring

```bash
# Install monitoring tools
apt-get install logwatch fail2ban

# Configure log rotation
cat > /etc/logrotate.d/task-flow << EOF
/var/www/task-flow/logs/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data www-data
}
EOF
```

### Performance Monitoring

```bash
# Install monitoring agent
curl -sSL https://monitoring.webvalue.com/install.sh | bash

# Configure monitoring
vim /etc/monitoring-agent.conf
```

## Backup Procedures

### Database Backup

```bash
#!/bin/bash
# backup-db.sh

DATE=$(date +%Y%m%d)
BACKUP_DIR="/var/www/task-flow/backup/database"
FILENAME="task_flow_${DATE}.sql.gz"

mysqldump -u backup_user -p'backup_password' task_flow | gzip > "${BACKUP_DIR}/${FILENAME}"

# Cleanup old backups (keep last 30 days)
find ${BACKUP_DIR} -type f -mtime +30 -delete
```

### File Backup

```bash
#!/bin/bash
# backup-files.sh

DATE=$(date +%Y%m%d)
BACKUP_DIR="/var/www/task-flow/backup/files"
FILENAME="task_flow_files_${DATE}.tar.gz"

tar -czf "${BACKUP_DIR}/${FILENAME}" -C /var/www/task-flow \
    --exclude='storage/cache/*' \
    --exclude='storage/logs/*' \
    .

# Cleanup old backups (keep last 7 days)
find ${BACKUP_DIR} -type f -mtime +7 -delete
```

## Maintenance

### Regular Tasks

```bash
# Add to crontab
0 1 * * * /var/www/task-flow/scripts/backup-db.sh
0 2 * * * /var/www/task-flow/scripts/backup-files.sh
0 3 * * * /var/www/task-flow/scripts/cleanup.sh
0 4 * * 0 /var/www/task-flow/scripts/optimize-db.sh
```

### Cleanup Script

```bash
#!/bin/bash
# cleanup.sh

# Clear temporary files
find /var/www/task-flow/storage/temp -type f -mtime +1 -delete

# Clear old logs
find /var/www/task-flow/logs -type f -mtime +30 -delete

# Clear cache
php /var/www/task-flow/clear-cache.php
```

## Support

For deployment support:
- Email: devops@webvalue.com
- Documentation: /docs/deployment
- Emergency: +1234567890

## Troubleshooting

Common issues and solutions:
1. Permission errors
2. Database connection issues
3. Cache problems
4. SSL certificate errors
5. Upload limitations

## Rollback Procedures

In case of deployment issues:
```bash
# Revert to previous version
git reset --hard HEAD^
git checkout v1.2.0

# Restore database
mysql task_flow < backup/database/task_flow_20240101.sql

# Clear cache
php clear-cache.php
