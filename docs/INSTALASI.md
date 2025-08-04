# Panduan Instalasi Lengkap

Dokumen ini berisi panduan instalasi lengkap untuk Aplikasi Survei Kepuasan Masyarakat (IKM).

## Daftar Isi

- [Persiapan Lingkungan](#persiapan-lingkungan)
- [Instalasi Dependencies](#instalasi-dependencies)
- [Konfigurasi Aplikasi](#konfigurasi-aplikasi)
- [Setup Database](#setup-database)
- [Konfigurasi Web Server](#konfigurasi-web-server)
- [Troubleshooting](#troubleshooting)

## Persiapan Lingkungan

### 1. Persyaratan Sistem

#### Server Development
- **OS**: Linux (Ubuntu 20.04+, CentOS 8+), macOS, atau Windows 10/11
- **PHP**: Version 8.1 atau lebih tinggi
- **Node.js**: Version 16.x atau lebih tinggi
- **Database**: MySQL 8.0+ atau MariaDB 10.3+
- **Web Server**: Apache 2.4+ atau Nginx 1.18+ (untuk production)

#### Server Production
- **RAM**: Minimum 2GB, disarankan 4GB+
- **Storage**: Minimum 5GB free space
- **CPU**: 2 cores minimum
- **Bandwidth**: Sesuai kebutuhan traffic

### 2. Instalasi PHP dan Extensions

#### Ubuntu/Debian
```bash
# Update package list
sudo apt update

# Install PHP dan extensions yang diperlukan
sudo apt install php8.1 php8.1-cli php8.1-common php8.1-mysql \
    php8.1-zip php8.1-gd php8.1-mbstring php8.1-curl php8.1-xml \
    php8.1-bcmath php8.1-fpm

# Verifikasi instalasi
php --version
```

#### CentOS/RHEL
```bash
# Install EPEL dan Remi repository
sudo dnf install epel-release
sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm

# Enable PHP 8.1
sudo dnf module enable php:remi-8.1

# Install PHP dan extensions
sudo dnf install php php-cli php-common php-mysqlnd php-zip \
    php-gd php-mbstring php-curl php-xml php-bcmath php-fpm
```

#### Windows
1. Download PHP 8.1+ dari [php.net](https://windows.php.net/download/)
2. Extract ke `C:\php`
3. Copy `php.ini-development` ke `php.ini`
4. Uncomment extensions yang diperlukan dalam `php.ini`
5. Tambahkan `C:\php` ke PATH environment variable

### 3. Instalasi Composer

#### Linux/macOS
```bash
# Download dan install Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# Verifikasi instalasi
composer --version
```

#### Windows
1. Download Composer installer dari [getcomposer.org](https://getcomposer.org/download/)
2. Jalankan installer dan ikuti petunjuk
3. Restart command prompt
4. Verifikasi dengan `composer --version`

### 4. Instalasi Node.js dan NPM

#### Linux (menggunakan NodeSource)
```bash
# Install Node.js 18.x
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verifikasi instalasi
node --version
npm --version
```

#### macOS
```bash
# Menggunakan Homebrew
brew install node

# Atau download installer dari nodejs.org
```

#### Windows
1. Download installer dari [nodejs.org](https://nodejs.org/)
2. Jalankan installer dan ikuti petunjuk
3. Restart command prompt
4. Verifikasi dengan `node --version` dan `npm --version`

### 5. Instalasi Database

#### MySQL
```bash
# Ubuntu/Debian
sudo apt install mysql-server mysql-client

# CentOS/RHEL
sudo dnf install mysql-server mysql

# Start dan enable service
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Secure installation
sudo mysql_secure_installation
```

#### MariaDB
```bash
# Ubuntu/Debian
sudo apt install mariadb-server mariadb-client

# CentOS/RHEL
sudo dnf install mariadb-server mariadb

# Start dan enable service
sudo systemctl start mariadb
sudo systemctl enable mariadb

# Secure installation
sudo mysql_secure_installation
```

## Instalasi Dependencies

### 1. Clone Repository

```bash
# Clone dari GitHub
git clone https://github.com/Chandra-INF-D/surver-kepuasan-diskominfo.git

# Masuk ke directory project
cd surver-kepuasan-diskominfo

# Cek branch yang tersedia
git branch -a
```

### 2. Install PHP Dependencies

```bash
# Install dependencies menggunakan Composer
composer install

# Untuk production (tanpa dev dependencies)
composer install --no-dev --optimize-autoloader
```

### 3. Install JavaScript Dependencies

```bash
# Install dependencies menggunakan NPM
npm install

# Atau menggunakan Yarn (jika tersedia)
yarn install
```

## Konfigurasi Aplikasi

### 1. Environment Configuration

```bash
# Copy file environment template
cp .env.example .env

# Generate application key
php artisan key:generate
```

### 2. Edit File .env

Edit file `.env` dan sesuaikan konfigurasi:

```env
# Application Settings
APP_NAME="Survei Kepuasan Masyarakat"
APP_ENV=local
APP_KEY=base64:generated-key-here
APP_DEBUG=true
APP_URL=http://localhost:8000

# Database Configuration
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=ikm_survey
DB_USERNAME=root
DB_PASSWORD=your_password

# Mail Configuration (untuk OTP)
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=your-email@gmail.com
MAIL_FROM_NAME="${APP_NAME}"

# Cache Configuration
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

# Additional Settings
LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug
```

### 3. Storage Setup

```bash
# Create storage link
php artisan storage:link

# Set proper permissions (Linux/macOS)
chmod -R 755 storage
chmod -R 755 bootstrap/cache
```

## Setup Database

### 1. Buat Database

```sql
-- Login ke MySQL/MariaDB
mysql -u root -p

-- Buat database
CREATE DATABASE ikm_survey CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Buat user khusus (opsional)
CREATE USER 'ikm_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON ikm_survey.* TO 'ikm_user'@'localhost';
FLUSH PRIVILEGES;

-- Exit
EXIT;
```

### 2. Jalankan Migrasi

```bash
# Jalankan migrasi database
php artisan migrate

# Cek status migrasi
php artisan migrate:status
```

### 3. Seeding Data (Opsional)

```bash
# Jalankan seeder untuk data awal
php artisan db:seed

# Atau jalankan seeder spesifik
php artisan db:seed --class=UserSeeder
```

## Build Assets

### Development Mode

```bash
# Build assets untuk development
npm run dev

# Atau dalam watch mode (auto-rebuild saat file berubah)
npm run dev -- --watch
```

### Production Mode

```bash
# Build assets untuk production (optimized)
npm run build
```

## Testing Installation

### 1. Jalankan Development Server

```bash
# Start Laravel development server
php artisan serve

# Aplikasi akan berjalan di http://localhost:8000
```

### 2. Verifikasi Fungsi Utama

1. **Akses Homepage**: Buka `http://localhost:8000`
2. **Test Database**: Cek apakah halaman loading tanpa error
3. **Test Assets**: Pastikan CSS dan JavaScript ter-load dengan benar
4. **Test Upload**: Coba fitur upload jika ada

### 3. Test Email Configuration

```bash
# Test email configuration
php artisan tinker

# Di dalam tinker console:
Mail::raw('Test email', function($message) {
    $message->to('test@example.com')->subject('Test');
});
```

## Konfigurasi Web Server

### Nginx Configuration

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/surver-kepuasan-diskominfo/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName your-domain.com
    DocumentRoot /var/www/surver-kepuasan-diskominfo/public

    <Directory /var/www/surver-kepuasan-diskominfo>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/ikm_error.log
    CustomLog ${APACHE_LOG_DIR}/ikm_access.log combined
</VirtualHost>
```

## Troubleshooting

### Common Issues

#### 1. Permission Errors
```bash
# Fix storage permissions
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache
```

#### 2. Composer Memory Limit
```bash
# Increase PHP memory limit temporarily
php -d memory_limit=2G /usr/local/bin/composer install
```

#### 3. Node.js Version Issues
```bash
# Using nvm to manage Node.js versions
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 18
nvm use 18
```

#### 4. Database Connection Errors
- Pastikan MySQL/MariaDB service berjalan
- Cek konfigurasi database di `.env`
- Pastikan user memiliki permission yang cukup
- Test koneksi manual: `mysql -h localhost -u username -p`

#### 5. Artisan Command Errors
```bash
# Clear cache
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear

# Regenerate autoload
composer dump-autoload
```

### Log Files

Monitor log files untuk debugging:

```bash
# Laravel logs
tail -f storage/logs/laravel.log

# Nginx logs
tail -f /var/log/nginx/error.log

# Apache logs
tail -f /var/log/apache2/error.log

# PHP-FPM logs
tail -f /var/log/php8.1-fpm.log
```

### Performance Optimization

```bash
# Cache configuration
php artisan config:cache

# Cache routes
php artisan route:cache

# Cache views
php artisan view:cache

# Optimize autoloader
composer install --optimize-autoloader --no-dev
```

## Backup dan Restore

### Database Backup
```bash
# Backup database
mysqldump -u username -p ikm_survey > backup_$(date +%Y%m%d_%H%M%S).sql

# Restore database
mysql -u username -p ikm_survey < backup_file.sql
```

### Files Backup
```bash
# Backup aplikasi (exclude vendor dan node_modules)
tar --exclude='vendor' --exclude='node_modules' --exclude='storage/logs' \
    -czf ikm_backup_$(date +%Y%m%d_%H%M%S).tar.gz surver-kepuasan-diskominfo/
```

---

Jika Anda mengalami masalah yang tidak tercakup dalam panduan ini, silakan buka [issue di GitHub](https://github.com/Chandra-INF-D/surver-kepuasan-diskominfo/issues) atau hubungi tim pengembang.