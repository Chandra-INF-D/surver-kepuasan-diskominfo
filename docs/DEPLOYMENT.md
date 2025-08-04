# Panduan Deployment

Dokumen ini berisi panduan lengkap untuk deployment Aplikasi Survei Kepuasan Masyarakat (IKM) ke berbagai environment.

## Daftar Isi

- [Persiapan Deployment](#persiapan-deployment)
- [Deployment ke Production Server](#deployment-ke-production-server)
- [Deployment dengan Docker](#deployment-dengan-docker)
- [Deployment ke Cloud Providers](#deployment-ke-cloud-providers)
- [CI/CD Pipeline](#cicd-pipeline)
- [Monitoring dan Maintenance](#monitoring-dan-maintenance)
- [Backup dan Recovery](#backup-dan-recovery)
- [Troubleshooting](#troubleshooting)

## Persiapan Deployment

### 1. Checklist Pre-Deployment

#### Code Quality
- [ ] Semua tests passing
- [ ] Code review completed
- [ ] Security scan completed
- [ ] Performance testing completed
- [ ] Documentation updated

#### Environment Setup
- [ ] Production server specifications met
- [ ] Database server configured
- [ ] SSL certificate installed
- [ ] Domain configured
- [ ] Email service configured
- [ ] Backup strategy implemented

#### Security Checklist
- [ ] Environment variables secured
- [ ] Database credentials rotated
- [ ] Firewall configured
- [ ] SSH key-based authentication
- [ ] Security headers configured
- [ ] Rate limiting implemented

### 2. Server Requirements

#### Minimum Requirements
- **CPU**: 2 cores
- **RAM**: 4GB
- **Storage**: 20GB SSD
- **Bandwidth**: 1Gbps
- **OS**: Ubuntu 20.04+ / CentOS 8+

#### Recommended Requirements
- **CPU**: 4 cores
- **RAM**: 8GB
- **Storage**: 50GB SSD
- **Bandwidth**: 1Gbps
- **OS**: Ubuntu 22.04 LTS

#### Software Requirements
- PHP 8.1+
- Nginx 1.18+ atau Apache 2.4+
- MySQL 8.0+ atau MariaDB 10.6+
- Redis 6.0+ (untuk caching dan sessions)
- Node.js 18+ (untuk asset building)
- Composer 2.0+

## Deployment ke Production Server

### 1. Server Setup

#### Update System
```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y

# CentOS/RHEL
sudo dnf update -y
```

#### Install Required Software
```bash
# Install PHP dan extensions
sudo apt install php8.1 php8.1-fpm php8.1-mysql php8.1-xml php8.1-curl \
    php8.1-gd php8.1-mbstring php8.1-zip php8.1-bcmath php8.1-intl

# Install Nginx
sudo apt install nginx

# Install MySQL
sudo apt install mysql-server

# Install Redis
sudo apt install redis-server

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs

# Install Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

### 2. Database Configuration

#### Create Database
```sql
# Login ke MySQL
sudo mysql -u root -p

# Create database
CREATE DATABASE ikm_survey CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

# Create dedicated user
CREATE USER 'ikm_user'@'localhost' IDENTIFIED BY 'secure_random_password';
GRANT ALL PRIVILEGES ON ikm_survey.* TO 'ikm_user'@'localhost';
FLUSH PRIVILEGES;

# Enable binary logging (for backups)
# Add to /etc/mysql/mysql.conf.d/mysqld.cnf
log-bin = /var/log/mysql/mysql-bin.log
binlog_do_db = ikm_survey

# Restart MySQL
sudo systemctl restart mysql
```

#### Secure MySQL Installation
```bash
sudo mysql_secure_installation
```

### 3. Application Deployment

#### Clone and Setup Application
```bash
# Create application directory
sudo mkdir -p /var/www/ikm-survey
cd /var/www/ikm-survey

# Clone repository
sudo git clone https://github.com/Chandra-INF-D/surver-kepuasan-diskominfo.git .

# Set ownership
sudo chown -R www-data:www-data /var/www/ikm-survey

# Switch to www-data user for remaining operations
sudo -u www-data bash

# Install PHP dependencies
composer install --no-dev --optimize-autoloader

# Install and build frontend assets
npm ci
npm run build

# Setup environment
cp .env.example .env
# Edit .env with production values

# Generate application key
php artisan key:generate

# Run migrations
php artisan migrate --force

# Cache optimization
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Set permissions
chmod -R 755 storage bootstrap/cache
```

#### Production .env Configuration
```env
APP_NAME="Survei Kepuasan Masyarakat"
APP_ENV=production
APP_KEY=base64:generated-key-here
APP_DEBUG=false
APP_URL=https://your-domain.com

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=error

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=ikm_survey
DB_USERNAME=ikm_user
DB_PASSWORD=secure_random_password

BROADCAST_DRIVER=log
CACHE_DRIVER=redis
FILESYSTEM_DISK=local
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis
SESSION_LIFETIME=120

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=your-email@gmail.com
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
AWS_USE_PATH_STYLE_ENDPOINT=false

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME=https
PUSHER_APP_CLUSTER=mt1

VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_HOST="${PUSHER_HOST}"
VITE_PUSHER_PORT="${PUSHER_PORT}"
VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

### 4. Web Server Configuration

#### Nginx Configuration
```nginx
# /etc/nginx/sites-available/ikm-survey
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;
    root /var/www/ikm-survey/public;

    # SSL Configuration
    ssl_certificate /etc/ssl/certs/your-domain.com.crt;
    ssl_certificate_key /etc/ssl/private/your-domain.com.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # Gzip Compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied expired no-cache no-store private auth;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    index index.php;

    charset utf-8;

    # Laravel specific
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    # PHP handling
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        
        # Security
        fastcgi_hide_header X-Powered-By;
        
        # Timeouts
        fastcgi_read_timeout 300;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
    }

    # Security: Block access to sensitive files
    location ~ /\.(?!well-known).* {
        deny all;
    }

    location ~ ^/(\.env|\.git|composer\.(json|lock)|package\.(json|lock)|yarn\.lock) {
        deny all;
    }

    # Cache static assets
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=60r/m;
    
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        try_files $uri $uri/ /index.php?$query_string;
    }
}
```

#### Enable Site
```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/ikm-survey /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

### 5. PHP-FPM Configuration

#### Configure PHP-FPM Pool
```ini
; /etc/php/8.1/fpm/pool.d/ikm-survey.conf
[ikm-survey]
user = www-data
group = www-data

listen = /var/run/php/php8.1-fpm-ikm.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660

pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
pm.max_requests = 500

; Environment variables
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp

; PHP configuration
php_value[session.save_handler] = files
php_value[session.save_path] = /var/lib/php/sessions
php_value[soap.wsdl_cache_dir] = /tmp
```

#### Restart PHP-FPM
```bash
sudo systemctl restart php8.1-fpm
```

### 6. SSL Certificate Setup

#### Using Let's Encrypt (Certbot)
```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Get certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Test auto-renewal
sudo certbot renew --dry-run
```

#### Using Custom Certificate
```bash
# Copy certificates
sudo cp your-domain.com.crt /etc/ssl/certs/
sudo cp your-domain.com.key /etc/ssl/private/

# Set permissions
sudo chmod 644 /etc/ssl/certs/your-domain.com.crt
sudo chmod 600 /etc/ssl/private/your-domain.com.key
```

## Deployment dengan Docker

### 1. Dockerfile

```dockerfile
# Multi-stage build
FROM node:18-alpine AS assets

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY resources/ resources/
COPY vite.config.js tailwind.config.js postcss.config.js ./
RUN npm run build

FROM php:8.1-fpm-alpine

# Install system dependencies
RUN apk add --no-cache \
    nginx \
    mysql-client \
    redis \
    supervisor \
    zip \
    unzip \
    git \
    curl

# Install PHP extensions
RUN docker-php-ext-install pdo pdo_mysql bcmath gd zip

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www/html

# Copy application files
COPY . .
COPY --from=assets /app/public/build public/build

# Install PHP dependencies
RUN composer install --no-dev --optimize-autoloader --no-interaction

# Set permissions
RUN chown -R www-data:www-data storage bootstrap/cache
RUN chmod -R 755 storage bootstrap/cache

# Copy configuration files
COPY docker/nginx.conf /etc/nginx/nginx.conf
COPY docker/php.ini /usr/local/etc/php/conf.d/app.ini
COPY docker/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

EXPOSE 80

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```

### 2. Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: ikm-app
    restart: unless-stopped
    ports:
      - "80:80"
    environment:
      - APP_ENV=production
      - DB_HOST=mysql
      - REDIS_HOST=redis
    volumes:
      - ./storage:/var/www/html/storage
      - ./bootstrap/cache:/var/www/html/bootstrap/cache
    depends_on:
      - mysql
      - redis
    networks:
      - ikm-network

  mysql:
    image: mysql:8.0
    container_name: ikm-mysql
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: ikm_survey
      MYSQL_USER: ikm_user
      MYSQL_PASSWORD: secure_password
      MYSQL_ROOT_PASSWORD: root_password
    volumes:
      - mysql_data:/var/lib/mysql
      - ./docker/mysql.cnf:/etc/mysql/conf.d/mysql.cnf
    ports:
      - "3306:3306"
    networks:
      - ikm-network

  redis:
    image: redis:7-alpine
    container_name: ikm-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - ikm-network

volumes:
  mysql_data:
  redis_data:

networks:
  ikm-network:
    driver: bridge
```

### 3. Deploy with Docker

```bash
# Build and start containers
docker-compose up -d

# Run migrations
docker-compose exec app php artisan migrate --force

# Setup application
docker-compose exec app php artisan key:generate
docker-compose exec app php artisan config:cache
docker-compose exec app php artisan route:cache
docker-compose exec app php artisan view:cache
```

## Deployment ke Cloud Providers

### 1. AWS EC2

#### Launch Instance
```bash
# Create EC2 instance
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1d0 \
    --count 1 \
    --instance-type t3.medium \
    --key-name your-key-pair \
    --security-group-ids sg-xxxxxxxxx \
    --subnet-id subnet-xxxxxxxxx \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=IKM-Survey-App}]'
```

#### Setup Load Balancer
```bash
# Create Application Load Balancer
aws elbv2 create-load-balancer \
    --name ikm-survey-alb \
    --subnets subnet-xxxxxxxxx subnet-yyyyyyyyy \
    --security-groups sg-xxxxxxxxx
```

#### RDS Setup
```bash
# Create RDS instance
aws rds create-db-instance \
    --db-instance-identifier ikm-survey-db \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --master-username admin \
    --master-user-password secure_password \
    --allocated-storage 20 \
    --vpc-security-group-ids sg-xxxxxxxxx
```

### 2. DigitalOcean Droplet

```bash
# Create droplet using doctl
doctl compute droplet create ikm-survey \
    --region sgp1 \
    --image ubuntu-22-04-x64 \
    --size s-2vcpu-4gb \
    --ssh-keys your-ssh-key-id
```

### 3. Google Cloud Platform

```bash
# Create VM instance
gcloud compute instances create ikm-survey-vm \
    --zone=asia-southeast1-a \
    --machine-type=e2-medium \
    --image-family=ubuntu-2204-lts \
    --image-project=ubuntu-os-cloud \
    --boot-disk-size=20GB
```

## CI/CD Pipeline

### 1. GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: password
          MYSQL_DATABASE: testing
        options: >-
          --health-cmd="mysqladmin ping"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=3
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.1'
        
    - name: Install Dependencies
      run: composer install
      
    - name: Run Tests
      run: php artisan test
      env:
        DB_HOST: 127.0.0.1
        DB_DATABASE: testing
        DB_USERNAME: root
        DB_PASSWORD: password

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy to server
      uses: appleboy/ssh-action@v0.1.5
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.KEY }}
        script: |
          cd /var/www/ikm-survey
          git pull origin main
          composer install --no-dev --optimize-autoloader
          npm ci && npm run build
          php artisan migrate --force
          php artisan config:cache
          php artisan route:cache
          php artisan view:cache
          sudo systemctl reload php8.1-fpm
          sudo systemctl reload nginx
```

### 2. GitLab CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  MYSQL_ROOT_PASSWORD: password
  MYSQL_DATABASE: testing

test:
  stage: test
  image: php:8.1
  services:
    - mysql:8.0
  before_script:
    - apt-get update -yqq
    - apt-get install -yqq git libzip-dev
    - docker-php-ext-install zip pdo_mysql
    - curl -sS https://getcomposer.org/installer | php
    - php composer.phar install
  script:
    - php artisan test
  
deploy_production:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
  script:
    - ssh -o StrictHostKeyChecking=no $USER@$HOST "cd /var/www/ikm-survey && ./deploy.sh"
  only:
    - main
```

### 3. Deployment Script

```bash
#!/bin/bash
# deploy.sh

set -e

echo "Starting deployment..."

# Backup current version
cp .env .env.backup
tar -czf "backup-$(date +%Y%m%d_%H%M%S).tar.gz" --exclude='vendor' --exclude='node_modules' .

# Pull latest code
git pull origin main

# Install dependencies
composer install --no-dev --optimize-autoloader --no-interaction

# Build assets
npm ci
npm run build

# Update database
php artisan migrate --force

# Clear and cache
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Restart services
sudo systemctl reload php8.1-fpm
sudo systemctl reload nginx

echo "Deployment completed successfully!"
```

## Monitoring dan Maintenance

### 1. Application Monitoring

#### Health Check Endpoint
```php
// routes/web.php
Route::get('/health', function () {
    $checks = [
        'database' => false,
        'cache' => false,
        'storage' => false,
    ];
    
    try {
        DB::connection()->getPdo();
        $checks['database'] = true;
    } catch (Exception $e) {
        // Database check failed
    }
    
    try {
        Cache::put('health_check', 'ok', 10);
        $checks['cache'] = Cache::get('health_check') === 'ok';
    } catch (Exception $e) {
        // Cache check failed
    }
    
    $checks['storage'] = is_writable(storage_path());
    
    $status = all($checks) ? 200 : 503;
    
    return response()->json([
        'status' => $status === 200 ? 'healthy' : 'unhealthy',
        'checks' => $checks,
        'timestamp' => now(),
    ], $status);
});
```

#### Log Monitoring
```bash
# Setup log rotation
sudo nano /etc/logrotate.d/ikm-survey

/var/www/ikm-survey/storage/logs/*.log {
    daily
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    copytruncate
}
```

### 2. Performance Monitoring

#### Setup New Relic (Optional)
```bash
# Install New Relic PHP agent
curl -L https://download.newrelic.com/548C16BF.gpg | sudo apt-key add -
echo "deb http://apt.newrelic.com/debian/ newrelic non-free" | sudo tee /etc/apt/sources.list.d/newrelic.list
sudo apt-get update
sudo apt-get install newrelic-php5
sudo newrelic-install install
```

#### Custom Metrics
```php
// Add to AppServiceProvider
public function boot()
{
    // Track response time
    app('log')->info('Request processed', [
        'url' => request()->url(),
        'method' => request()->method(),
        'response_time' => microtime(true) - LARAVEL_START,
        'memory_usage' => memory_get_peak_usage(true),
    ]);
}
```

### 3. Automated Backups

#### Database Backup Script
```bash
#!/bin/bash
# backup-db.sh

BACKUP_DIR="/var/backups/ikm-survey"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="ikm_survey"
DB_USER="ikm_user"
DB_PASS="secure_password"

mkdir -p $BACKUP_DIR

# Create database backup
mysqldump -u $DB_USER -p$DB_PASS $DB_NAME | gzip > "$BACKUP_DIR/db_backup_$DATE.sql.gz"

# Remove backups older than 30 days
find $BACKUP_DIR -name "db_backup_*.sql.gz" -mtime +30 -delete

echo "Database backup completed: db_backup_$DATE.sql.gz"
```

#### Setup Cron Job
```bash
# Add to crontab
0 2 * * * /var/www/ikm-survey/scripts/backup-db.sh
0 3 * * 0 /var/www/ikm-survey/scripts/backup-files.sh
```

## Backup dan Recovery

### 1. Database Backup dan Restore

#### Backup
```bash
# Full backup
mysqldump -u ikm_user -p ikm_survey > backup_$(date +%Y%m%d_%H%M%S).sql

# Backup with compression
mysqldump -u ikm_user -p ikm_survey | gzip > backup_$(date +%Y%m%d_%H%M%S).sql.gz

# Backup specific tables
mysqldump -u ikm_user -p ikm_survey respondens answers > backup_responses.sql
```

#### Restore
```bash
# Restore from backup
mysql -u ikm_user -p ikm_survey < backup_20241201_020000.sql

# Restore from compressed backup
gunzip < backup_20241201_020000.sql.gz | mysql -u ikm_user -p ikm_survey
```

### 2. File System Backup

```bash
#!/bin/bash
# backup-files.sh

BACKUP_DIR="/var/backups/ikm-survey"
APP_DIR="/var/www/ikm-survey"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Backup application files (exclude vendor and node_modules)
tar --exclude='vendor' --exclude='node_modules' --exclude='storage/logs' \
    -czf "$BACKUP_DIR/files_backup_$DATE.tar.gz" -C $APP_DIR .

# Remove backups older than 7 days
find $BACKUP_DIR -name "files_backup_*.tar.gz" -mtime +7 -delete

echo "Files backup completed: files_backup_$DATE.tar.gz"
```

### 3. Automated Cloud Backup

#### AWS S3 Backup
```bash
#!/bin/bash
# backup-to-s3.sh

S3_BUCKET="ikm-survey-backups"
LOCAL_BACKUP_DIR="/var/backups/ikm-survey"

# Upload to S3
aws s3 sync $LOCAL_BACKUP_DIR s3://$S3_BUCKET/$(date +%Y/%m)/ \
    --delete \
    --storage-class STANDARD_IA
```

## Troubleshooting

### 1. Common Issues

#### Permission Issues
```bash
# Fix storage permissions
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 755 storage bootstrap/cache

# Fix SELinux issues (CentOS/RHEL)
sudo setsebool -P httpd_can_network_connect 1
sudo setsebool -P httpd_execmem 1
```

#### Database Connection Issues
```bash
# Test database connection
php artisan tinker
DB::connection()->getPdo();

# Check MySQL status
sudo systemctl status mysql
sudo tail -f /var/log/mysql/error.log
```

#### PHP-FPM Issues
```bash
# Check PHP-FPM status
sudo systemctl status php8.1-fpm

# Check PHP-FPM logs
sudo tail -f /var/log/php8.1-fpm.log

# Restart PHP-FPM
sudo systemctl restart php8.1-fpm
```

### 2. Performance Issues

#### Debug Slow Queries
```bash
# Enable MySQL slow query log
# Add to /etc/mysql/mysql.conf.d/mysqld.cnf
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2

# Restart MySQL
sudo systemctl restart mysql

# Analyze slow queries
sudo pt-query-digest /var/log/mysql/slow.log
```

#### Monitor System Resources
```bash
# Check system resources
htop
iostat -x 1
free -h
df -h

# Monitor PHP-FPM pool
sudo tail -f /var/log/php8.1-fpm.log | grep "WARNING\|ERROR"
```

### 3. Rollback Procedures

#### Application Rollback
```bash
#!/bin/bash
# rollback.sh

BACKUP_FILE=$1

if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: ./rollback.sh backup_20241201_020000.tar.gz"
    exit 1
fi

echo "Rolling back to $BACKUP_FILE..."

# Stop services
sudo systemctl stop nginx php8.1-fpm

# Backup current state
mv /var/www/ikm-survey /var/www/ikm-survey.backup.$(date +%Y%m%d_%H%M%S)

# Restore from backup
mkdir -p /var/www/ikm-survey
tar -xzf /var/backups/ikm-survey/$BACKUP_FILE -C /var/www/ikm-survey

# Set permissions
sudo chown -R www-data:www-data /var/www/ikm-survey
sudo chmod -R 755 /var/www/ikm-survey/storage /var/www/ikm-survey/bootstrap/cache

# Start services
sudo systemctl start php8.1-fpm nginx

echo "Rollback completed successfully!"
```

#### Database Rollback
```bash
# Backup current database
mysqldump -u ikm_user -p ikm_survey > current_backup_$(date +%Y%m%d_%H%M%S).sql

# Restore from backup
mysql -u ikm_user -p ikm_survey < backup_20241201_020000.sql
```

---

Deployment yang sukses memerlukan perencanaan yang matang dan testing yang thorough. Pastikan untuk selalu melakukan testing di staging environment sebelum deployment ke production, dan selalu siapkan strategi rollback jika terjadi masalah.