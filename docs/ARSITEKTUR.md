# Arsitektur Aplikasi

Dokumen ini menjelaskan arsitektur dan komponen utama dari Aplikasi Survei Kepuasan Masyarakat (IKM).

## Daftar Isi

- [Gambaran Umum Arsitektur](#gambaran-umum-arsitektur)
- [Arsitektur MVC Laravel](#arsitektur-mvc-laravel)
- [Database Schema](#database-schema)
- [Komponen Backend](#komponen-backend)
- [Komponen Frontend](#komponen-frontend)
- [Alur Data](#alur-data)
- [Security Architecture](#security-architecture)
- [Performance & Scalability](#performance--scalability)

## Gambaran Umum Arsitektur

Aplikasi IKM menggunakan arsitektur monolitik berbasis Laravel framework dengan pola MVC (Model-View-Controller). Berikut adalah diagram high-level arsitektur:

```
┌─────────────────────────────────────────────────┐
│                   Frontend                      │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐│
│  │ Public View │ │ Admin Panel │ │ API Clients ││
│  └─────────────┘ └─────────────┘ └─────────────┘│
└─────────────────────┬───────────────────────────┘
                      │ HTTP Requests
                      ▼
┌─────────────────────────────────────────────────┐
│                Web Server                       │
│              (Nginx/Apache)                     │
└─────────────────────┬───────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────┐
│               Laravel Application               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐│
│  │ Controllers │ │   Models    │ │    Views    ││
│  └─────────────┘ └─────────────┘ └─────────────┘│
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐│
│  │ Middleware  │ │  Services   │ │   Helpers   ││
│  └─────────────┘ └─────────────┘ └─────────────┘│
└─────────────────────┬───────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────┐
│                  Database                       │
│               (MySQL/MariaDB)                   │
└─────────────────────────────────────────────────┘
```

## Arsitektur MVC Laravel

### Model Layer
Model layer menangani logika bisnis dan interaksi database menggunakan Eloquent ORM.

#### Core Models
```php
// app/Models/
├── User.php           # Model untuk pengguna (admin)
├── Kuesioner.php      # Model untuk pertanyaan survei
├── Responden.php      # Model untuk responden survei
├── Answer.php         # Model untuk jawaban responden
├── Unsur.php          # Model untuk unsur penilaian
├── Village.php        # Model untuk wilayah/desa
├── SatkerType.php     # Model untuk tipe satuan kerja
└── Feedback.php       # Model untuk feedback/saran
```

#### Model Relationships
```php
// Relasi antar model
User → Village (belongsTo)
Village → SatkerType (belongsTo)
Kuesioner → Village (belongsTo)
Kuesioner → Unsur (belongsTo)
Responden → Village (belongsTo)
Responden → Answer (hasMany)
Answer → Kuesioner (belongsTo)
Answer → Responden (belongsTo)
```

### Controller Layer
Controller menangani HTTP requests dan koordinasi antara model dan view.

#### Public Controllers
- `IndexController`: Halaman publik dan form survei
- `AuthController`: Autentikasi dan login
- `OtpController`: Verifikasi OTP

#### Admin Controllers
- `DasborController`: Dashboard admin dan analisis IKM
- `KuesionerController`: CRUD kuesioner
- `RespondenController`: Manajemen responden
- `FeedbackController`: Manajemen feedback
- `ExportController`: Export data ke Excel/PDF
- `ProfilController`: Manajemen profil admin

### View Layer
View menggunakan Blade template engine dengan Tailwind CSS untuk styling.

#### View Structure
```
resources/views/
├── layouts/
│   ├── main.blade.php      # Layout untuk halaman publik
│   └── dasbor.blade.php    # Layout untuk admin panel
├── components/             # Reusable components
├── public/                 # Halaman publik
│   ├── index.blade.php     # Homepage
│   └── kuesioner.blade.php # Form survei
└── dasbor/                 # Admin panel views
    ├── index.blade.php     # Dashboard
    ├── kuesioner/          # Manajemen kuesioner
    ├── responden/          # Manajemen responden
    ├── feedback/           # Manajemen feedback
    └── profil/             # Profil admin
```

## Database Schema

### ERD (Entity Relationship Diagram)

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ SatkerType  │────│   Village   │────│    User     │
│             │    │             │    │             │
│ - id        │    │ - id        │    │ - id        │
│ - name      │    │ - name      │    │ - name      │
│ - created_at│    │ - satker_id │    │ - email     │
│ - updated_at│    │ - created_at│    │ - village_id│
└─────────────┘    │ - updated_at│    │ - role      │
                   └─────────────┘    │ - created_at│
                           │          │ - updated_at│
                           │          └─────────────┘
                           │
        ┌─────────────┐    │    ┌─────────────┐
        │   Unsur     │    │    │ Kuesioner   │
        │             │    │    │             │
        │ - id        │    └────│ - id        │
        │ - name      │         │ - village_id│
        │ - created_at│    ┌────│ - unsur_id  │
        │ - updated_at│    │    │ - question  │
        └─────────────┘    │    │ - is_active │
                           │    │ - uuid      │
        ┌─────────────┐    │    │ - created_at│
        │  Responden  │    │    │ - updated_at│
        │             │    │    └─────────────┘
        │ - id        │    │            │
        │ - village_id│────┘            │
        │ - name      │                 │
        │ - email     │                 │
        │ - phone     │                 │
        │ - uuid      │    ┌─────────────┐
        │ - created_at│    │   Answer    │
        │ - updated_at│    │             │
        └─────────────┘    │ - id        │
                │          │ - responden_│
                └──────────│ - kuesioner_│
                           │ - answer    │
                           │ - created_at│
                           │ - updated_at│
                           └─────────────┘
```

### Key Tables

#### 1. users
```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role ENUM('admin_utama', 'admin_satker') DEFAULT 'admin_satker',
    village_id BIGINT,
    email_verified_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (village_id) REFERENCES villages(id)
);
```

#### 2. kuesioners
```sql
CREATE TABLE kuesioners (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    village_id BIGINT,
    unsur_id BIGINT,
    question TEXT NOT NULL,
    is_active BOOLEAN DEFAULT true,
    uuid VARCHAR(36) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (village_id) REFERENCES villages(id),
    FOREIGN KEY (unsur_id) REFERENCES unsurs(id)
);
```

#### 3. respondens
```sql
CREATE TABLE respondens (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    village_id BIGINT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    phone VARCHAR(255),
    uuid VARCHAR(36) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (village_id) REFERENCES villages(id)
);
```

#### 4. answers
```sql
CREATE TABLE answers (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    responden_id BIGINT,
    kuesioner_id BIGINT,
    answer TINYINT NOT NULL CHECK (answer BETWEEN 1 AND 4),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (responden_id) REFERENCES respondens(id) ON DELETE CASCADE,
    FOREIGN KEY (kuesioner_id) REFERENCES kuesioners(id) ON DELETE CASCADE
);
```

## Komponen Backend

### 1. Authentication & Authorization

#### Authentication Flow
```php
// AuthController.php
1. User input email/password
2. Validate credentials
3. Generate OTP and send via email
4. User input OTP
5. Verify OTP and create session
6. Redirect to dashboard
```

#### Authorization Middleware
```php
// Role-based access control
'admin_utama' → Full access to all features
'admin_satker' → Limited to specific village/satker
```

### 2. Business Logic Services

#### IKM Calculation Engine
Algoritma perhitungan IKM (dalam `app/helpers.php`):

```php
function getIKM($respondens, $kuesioners) {
    // 1. Hitung bobot nilai tertimbang
    $bobotNilaiTertimbang = 1 / count($kuesioners);
    
    // 2. Hitung total nilai persepsi per unit
    // 3. Hitung NRR (Nilai Rata-rata per Unsur)
    // 4. Hitung NRR Tertimbang
    // 5. Hitung IKM = Σ(NRR Tertimbang)
    // 6. Konversi IKM = IKM × 25
    
    return [
        'data' => $data,
        'IKM' => $IKM,
        'konversiIKM' => $konversiIKM,
        'bobotNilaiTertimbang' => $bobotNilaiTertimbang
    ];
}
```

#### Export Services
```php
// app/Exports/
├── IkmExport.php          # Export data IKM ke Excel
├── RespondenExport.php    # Export data responden
└── FeedbackExport.php     # Export feedback
```

### 3. Email System

#### OTP Email
```php
// app/Mail/OtpMail.php
class OtpMail extends Mailable {
    public function build() {
        return $this->view('emails.otp')
                   ->subject('Kode OTP untuk Login');
    }
}
```

### 4. File Upload & Storage

Menggunakan Laravel Storage untuk mengelola file uploads:
```php
// Storage configuration
'public' => [
    'driver' => 'local',
    'root' => storage_path('app/public'),
    'url' => env('APP_URL').'/storage',
    'visibility' => 'public',
]
```

## Komponen Frontend

### 1. Asset Pipeline

#### Vite Configuration
```javascript
// vite.config.js
export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```

#### CSS Framework
- **Tailwind CSS**: Utility-first CSS framework
- **Flowbite**: UI components library
- **Custom CSS**: Additional styling in `resources/css/app.css`

#### JavaScript Libraries
- **Alpine.js**: Minimal framework for JavaScript behavior
- **Tom Select**: Enhanced select boxes
- **Chart.js**: Data visualization (untuk dashboard)

### 2. UI Components

#### Reusable Blade Components
```php
// resources/views/components/
├── alert.blade.php        # Alert notifications
├── button.blade.php       # Styled buttons
├── input.blade.php        # Form inputs
├── modal.blade.php        # Modal dialogs
└── table.blade.php        # Data tables
```

### 3. Form Handling

#### Survey Form (Public)
```javascript
// Validasi form survei
document.getElementById('survey-form').addEventListener('submit', function(e) {
    // Validate all questions answered
    // Show confirmation modal
    // Submit form
});
```

#### Admin Forms
- CRUD forms dengan validation
- File upload handling
- AJAX form submissions untuk UX yang lebih baik

## Alur Data

### 1. Survey Response Flow

```
1. Responden → Akses halaman survei
2. System → Load kuesioner aktif untuk wilayah
3. Responden → Isi data dan jawaban
4. System → Validate data dan simpan ke database
5. System → Generate UUID untuk tracking
6. System → Tampilkan konfirmasi
```

### 2. IKM Calculation Flow

```
1. Admin → Request analisis IKM
2. System → Ambil data responden dan jawaban
3. System → Jalankan algoritma perhitungan IKM
4. System → Generate laporan dan visualisasi
5. Admin → Export atau preview hasil
```

### 3. Export Process Flow

```
1. Admin → Pilih data untuk export
2. System → Generate query berdasarkan filter
3. System → Process data menggunakan Excel/PDF library
4. System → Generate file dan provide download link
5. Admin → Download file
```

## Security Architecture

### 1. Authentication Security
- Password hashing menggunakan bcrypt
- OTP verification untuk additional security
- Session management dengan Laravel guards
- CSRF protection pada semua forms

### 2. Authorization
- Role-based access control (RBAC)
- Route protection dengan middleware
- Data isolation berdasarkan village/satker

### 3. Data Protection
- Input validation dan sanitization
- SQL injection protection via Eloquent ORM
- XSS protection dengan Blade escaping
- File upload validation dan restrictions

### 4. Security Headers
```php
// Middleware untuk security headers
X-Frame-Options: SAMEORIGIN
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000
```

## Performance & Scalability

### 1. Database Optimization
- Proper indexing pada foreign keys
- Query optimization dengan Eloquent relationships
- Database connection pooling

### 2. Caching Strategy
```php
// Config caching
php artisan config:cache
php artisan route:cache
php artisan view:cache

// Application caching
Cache::remember('ikm_data', 3600, function() {
    return $this->calculateIKM();
});
```

### 3. Asset Optimization
- CSS/JS minification dengan Vite
- Image optimization
- Lazy loading untuk large datasets

### 4. Scalability Considerations
- Horizontal scaling dengan load balancer
- Database read replicas untuk reporting
- CDN untuk static assets
- Queue system untuk heavy processing

## Monitoring & Logging

### 1. Application Logs
```php
// Laravel logging
Log::info('Survey submitted', ['responden_id' => $id]);
Log::error('Export failed', ['error' => $exception->getMessage()]);
```

### 2. Performance Monitoring
- Database query monitoring
- Response time tracking
- Memory usage monitoring
- Error rate monitoring

### 3. Health Checks
```php
// Route untuk health check
Route::get('/health', function() {
    return response()->json([
        'status' => 'ok',
        'database' => DB::connection()->getPdo() ? 'connected' : 'disconnected',
        'cache' => Cache::store()->getStore() ? 'available' : 'unavailable'
    ]);
});
```

---

Arsitektur ini dirancang untuk memberikan fleksibilitas, maintainability, dan scalability yang baik untuk aplikasi survei kepuasan masyarakat yang dapat berkembang seiring dengan kebutuhan organisasi.