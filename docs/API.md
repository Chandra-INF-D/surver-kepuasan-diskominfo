# Dokumentasi API dan Fitur

Dokumen ini menjelaskan endpoint API, fitur utama, dan cara penggunaannya dalam Aplikasi Survei Kepuasan Masyarakat (IKM).

## Daftar Isi

- [Authentication](#authentication)
- [Public Endpoints](#public-endpoints)
- [Admin Endpoints](#admin-endpoints)
- [Data Models](#data-models)
- [Response Format](#response-format)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [Examples](#examples)

## Authentication

Aplikasi menggunakan session-based authentication dengan verifikasi OTP.

### Login Process

#### 1. Initial Login
```http
POST /auth/login
Content-Type: application/json

{
    "email": "admin@example.com",
    "password": "password123"
}
```

**Response:**
```json
{
    "success": true,
    "message": "OTP telah dikirim ke email Anda",
    "data": {
        "email": "admin@example.com",
        "otp_sent": true
    }
}
```

#### 2. OTP Verification
```http
POST /otp/verify
Content-Type: application/json

{
    "email": "admin@example.com",
    "otp": "123456"
}
```

**Response:**
```json
{
    "success": true,
    "message": "Login berhasil",
    "data": {
        "user": {
            "id": 1,
            "name": "Admin User",
            "email": "admin@example.com",
            "role": "admin_utama"
        },
        "redirect_url": "/dasbor"
    }
}
```

#### 3. Logout
```http
GET /auth/logout
```

**Response:**
```json
{
    "success": true,
    "message": "Logout berhasil"
}
```

## Public Endpoints

### Homepage
```http
GET /
```
Menampilkan halaman utama aplikasi dengan informasi survei yang tersedia.

### Survey Form
```http
GET /kuesioner
```
Menampilkan form kuesioner untuk diisi responden.

### Submit Survey Response
```http
POST /result/store
Content-Type: application/json

{
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "08123456789",
    "village_id": 1,
    "answers": [
        {
            "kuesioner_id": 1,
            "answer": 4
        },
        {
            "kuesioner_id": 2,
            "answer": 3
        }
    ]
}
```

**Response:**
```json
{
    "success": true,
    "message": "Terima kasih atas partisipasi Anda",
    "data": {
        "responden_uuid": "550e8400-e29b-41d4-a716-446655440000",
        "total_answers": 2
    }
}
```

## Admin Endpoints

Semua endpoint admin memerlukan autentikasi.

### Dashboard

#### Get Dashboard Data
```http
GET /dasbor
```

**Response:** HTML dashboard dengan statistik dan grafik.

#### Get IKM Analysis
```http
GET /dasbor/ikm
```

**Query Parameters:**
- `village_id` (optional): Filter by village
- `start_date` (optional): Start date filter
- `end_date` (optional): End date filter

**Response:** HTML halaman analisis IKM.

### Kuesioner Management

#### List Kuesioner
```http
GET /dasbor/kuesioner
```

**Query Parameters:**
- `search` (optional): Search term
- `village_id` (optional): Filter by village
- `is_active` (optional): Filter by status

#### Create Kuesioner
```http
POST /dasbor/kuesioner
Content-Type: application/json

{
    "village_id": 1,
    "unsur_id": 1,
    "question": "Bagaimana penilaian Anda terhadap layanan?",
    "is_active": true
}
```

#### Show Kuesioner
```http
GET /dasbor/kuesioner/{uuid}
```

#### Update Kuesioner
```http
PATCH /dasbor/kuesioner/{uuid}
Content-Type: application/json

{
    "question": "Updated question text",
    "is_active": false
}
```

#### Delete Kuesioner
```http
DELETE /dasbor/kuesioner/{uuid}
```

#### Bulk Operations
```http
POST /dasbor/kuesioner/checks
Content-Type: application/json

{
    "action": "activate|deactivate|delete",
    "ids": [1, 2, 3]
}
```

### Responden Management

#### List Responden
```http
GET /dasbor/responden
```

**Query Parameters:**
- `search` (optional): Search by name or email
- `village_id` (optional): Filter by village
- `start_date` (optional): Start date filter
- `end_date` (optional): End date filter

#### Show Responden Detail
```http
GET /dasbor/responden/{uuid}
```

**Response:** HTML detail responden dengan jawaban survei.

### Feedback Management

#### List Feedback
```http
GET /dasbor/feedback
```

**Query Parameters:**
- `search` (optional): Search in feedback content
- `village_id` (optional): Filter by village

### Export Functions

#### Export IKM Data
```http
GET /dasbor/ikm/export
```

**Query Parameters:**
- `format`: `excel|pdf`
- `village_id` (optional): Filter by village
- `start_date` (optional): Start date filter
- `end_date` (optional): End date filter

**Response:** File download

#### Preview IKM Report
```http
GET /dasbor/ikm/preview
```

**Query Parameters:** Same as export

**Response:** HTML preview of the report

#### Export Responden Data
```http
GET /dasbor/laporan/responden/export
```

**Query Parameters:**
- `format`: `excel|pdf`
- `village_id` (optional): Filter by village
- `start_date` (optional): Start date filter
- `end_date` (optional): End date filter

#### Export Feedback Data
```http
GET /dasbor/laporan/feedback/export/table
```

**Query Parameters:**
- `format`: `excel|pdf`
- `village_id` (optional): Filter by village

### Profile Management

#### Show Profile
```http
GET /dasbor/profil
```

#### Update Profile
```http
PATCH /dasbor/profil
Content-Type: application/json

{
    "name": "Updated Name",
    "email": "updated@example.com"
}
```

#### Change Password
```http
POST /dasbor/change-password
Content-Type: application/json

{
    "current_password": "current123",
    "password": "newpassword123",
    "password_confirmation": "newpassword123"
}
```

## Data Models

### User
```json
{
    "id": 1,
    "name": "Admin User",
    "email": "admin@example.com",
    "role": "admin_utama|admin_satker",
    "village_id": 1,
    "email_verified_at": "2024-01-01T00:00:00Z",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z"
}
```

### Village
```json
{
    "id": 1,
    "name": "Desa Contoh",
    "satker_type_id": 1,
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z"
}
```

### SatkerType
```json
{
    "id": 1,
    "name": "Pemerintahan Desa",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z"
}
```

### Unsur
```json
{
    "id": 1,
    "name": "Persyaratan",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z"
}
```

### Kuesioner
```json
{
    "id": 1,
    "village_id": 1,
    "unsur_id": 1,
    "question": "Bagaimana penilaian Anda terhadap persyaratan layanan?",
    "is_active": true,
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z",
    "village": {
        "id": 1,
        "name": "Desa Contoh"
    },
    "unsur": {
        "id": 1,
        "name": "Persyaratan"
    }
}
```

### Responden
```json
{
    "id": 1,
    "village_id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "08123456789",
    "uuid": "550e8400-e29b-41d4-a716-446655440001",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z",
    "village": {
        "id": 1,
        "name": "Desa Contoh"
    }
}
```

### Answer
```json
{
    "id": 1,
    "responden_id": 1,
    "kuesioner_id": 1,
    "answer": 4,
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z",
    "responden": {
        "id": 1,
        "name": "John Doe"
    },
    "kuesioner": {
        "id": 1,
        "question": "Bagaimana penilaian Anda terhadap persyaratan layanan?"
    }
}
```

### Feedback
```json
{
    "id": 1,
    "village_id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "feedback": "Layanan sudah cukup baik, namun perlu peningkatan di bagian...",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z",
    "village": {
        "id": 1,
        "name": "Desa Contoh"
    }
}
```

## Response Format

### Success Response
```json
{
    "success": true,
    "message": "Operation completed successfully",
    "data": {
        // Response data here
    }
}
```

### Error Response
```json
{
    "success": false,
    "message": "Error message",
    "errors": {
        "field_name": ["Error detail 1", "Error detail 2"]
    },
    "error_code": "VALIDATION_ERROR"
}
```

### Pagination Response
```json
{
    "success": true,
    "data": {
        "data": [
            // Array of items
        ],
        "current_page": 1,
        "first_page_url": "http://example.com/api/endpoint?page=1",
        "from": 1,
        "last_page": 5,
        "last_page_url": "http://example.com/api/endpoint?page=5",
        "next_page_url": "http://example.com/api/endpoint?page=2",
        "path": "http://example.com/api/endpoint",
        "per_page": 15,
        "prev_page_url": null,
        "to": 15,
        "total": 75
    }
}
```

## Error Handling

### HTTP Status Codes

- `200` - OK (Success)
- `201` - Created (Resource created successfully)
- `400` - Bad Request (Invalid request data)
- `401` - Unauthorized (Authentication required)
- `403` - Forbidden (Access denied)
- `404` - Not Found (Resource not found)
- `422` - Unprocessable Entity (Validation errors)
- `500` - Internal Server Error (Server error)

### Error Types

#### Validation Errors (422)
```json
{
    "success": false,
    "message": "The given data was invalid.",
    "errors": {
        "email": ["The email field is required."],
        "password": ["The password must be at least 8 characters."]
    },
    "error_code": "VALIDATION_ERROR"
}
```

#### Authentication Errors (401)
```json
{
    "success": false,
    "message": "Unauthenticated.",
    "error_code": "AUTHENTICATION_ERROR"
}
```

#### Authorization Errors (403)
```json
{
    "success": false,
    "message": "Access denied. Insufficient permissions.",
    "error_code": "AUTHORIZATION_ERROR"
}
```

#### Not Found Errors (404)
```json
{
    "success": false,
    "message": "Resource not found.",
    "error_code": "NOT_FOUND"
}
```

## Rate Limiting

Aplikasi mengimplementasikan rate limiting untuk mencegah spam dan abuse:

- **Public endpoints**: 60 requests per minute per IP
- **Admin endpoints**: 120 requests per minute per authenticated user
- **Survey submission**: 5 submissions per minute per IP

### Rate Limit Headers
```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
X-RateLimit-Reset: 1640995200
```

## Examples

### Complete Survey Submission Flow

#### 1. Get Available Surveys
```bash
curl -X GET "https://example.com/kuesioner" \
     -H "Accept: text/html"
```

#### 2. Submit Survey Response
```bash
curl -X POST "https://example.com/result/store" \
     -H "Content-Type: application/json" \
     -H "X-CSRF-TOKEN: your-csrf-token" \
     -d '{
         "name": "John Doe",
         "email": "john@example.com",
         "phone": "08123456789",
         "village_id": 1,
         "answers": [
             {"kuesioner_id": 1, "answer": 4},
             {"kuesioner_id": 2, "answer": 3},
             {"kuesioner_id": 3, "answer": 4}
         ]
     }'
```

### Admin Login and Management

#### 1. Login
```bash
curl -X POST "https://example.com/auth/login" \
     -H "Content-Type: application/json" \
     -H "X-CSRF-TOKEN: your-csrf-token" \
     -d '{
         "email": "admin@example.com",
         "password": "password123"
     }'
```

#### 2. Verify OTP
```bash
curl -X POST "https://example.com/otp/verify" \
     -H "Content-Type: application/json" \
     -H "X-CSRF-TOKEN: your-csrf-token" \
     -d '{
         "email": "admin@example.com",
         "otp": "123456"
     }'
```

#### 3. Create New Question
```bash
curl -X POST "https://example.com/dasbor/kuesioner" \
     -H "Content-Type: application/json" \
     -H "X-CSRF-TOKEN: your-csrf-token" \
     -b "session-cookie" \
     -d '{
         "village_id": 1,
         "unsur_id": 1,
         "question": "Bagaimana penilaian Anda terhadap kecepatan pelayanan?",
         "is_active": true
     }'
```

### Export Data

#### 1. Export IKM to Excel
```bash
curl -X GET "https://example.com/dasbor/ikm/export?format=excel&village_id=1&start_date=2024-01-01&end_date=2024-12-31" \
     -H "Accept: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet" \
     -b "session-cookie" \
     -o "ikm_report.xlsx"
```

#### 2. Export Responden to PDF
```bash
curl -X GET "https://example.com/dasbor/laporan/responden/export?format=pdf&village_id=1" \
     -H "Accept: application/pdf" \
     -b "session-cookie" \
     -o "responden_report.pdf"
```

## Helper Functions

### IKM Calculation
Aplikasi menyediakan fungsi helper untuk perhitungan IKM:

```php
/**
 * Calculate IKM based on responses and questionnaires
 *
 * @param Collection $respondens
 * @param Collection $kuesioners
 * @return array
 */
function getIKM($respondens, $kuesioners)
{
    // Implementation in app/helpers.php
    return [
        'data' => $data,              // Detailed calculation per question
        'IKM' => $IKM,                // Raw IKM value (0-4)
        'konversiIKM' => $konversiIKM, // Converted IKM value (0-100)
        'bobotNilaiTertimbang' => $bobotNilaiTertimbang // Weight per question
    ];
}
```

### Rating Labels
```php
/**
 * Get rating label based on numeric value
 *
 * @param int $rating
 * @return string
 */
function rateLabel($rating)
{
    // 1: "Tidak Baik"
    // 2: "Kurang Baik"
    // 3: "Baik"
    // 4: "Sangat Baik"
}
```

### Performance Assessment
```php
/**
 * Get performance assessment based on IKM conversion
 *
 * @param float $konversiIKM
 * @return object
 */
function nilaPersepsi($konversiIKM)
{
    // Returns object with 'mutu' (A/B/C/D) and 'kinerja' (performance level)
}
```

---

Dokumentasi ini akan terus diperbarui seiring dengan pengembangan fitur baru. Untuk pertanyaan atau bantuan, silakan hubungi tim pengembang atau buka issue di repository GitHub.