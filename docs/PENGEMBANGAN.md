# Panduan Pengembangan

Dokumen ini berisi panduan lengkap untuk pengembangan Aplikasi Survei Kepuasan Masyarakat (IKM).

## Daftar Isi

- [Setup Development Environment](#setup-development-environment)
- [Development Workflow](#development-workflow)
- [Coding Standards](#coding-standards)
- [Database Development](#database-development)
- [Testing](#testing)
- [Debugging](#debugging)
- [Performance Optimization](#performance-optimization)
- [Deployment](#deployment)

## Setup Development Environment

### 1. Prerequisites

Pastikan Anda telah menginstall:
- PHP 8.1+
- Composer
- Node.js 16+
- MySQL/MariaDB
- Git

### 2. Clone dan Setup Project

```bash
# Clone repository
git clone https://github.com/Chandra-INF-D/surver-kepuasan-diskominfo.git
cd surver-kepuasan-diskominfo

# Install dependencies
composer install
npm install

# Setup environment
cp .env.example .env
php artisan key:generate

# Setup database
php artisan migrate
php artisan db:seed

# Build assets
npm run dev
```

### 3. IDE Configuration

#### VS Code Extensions (Recommended)
```json
{
    "recommendations": [
        "bmewburn.vscode-intelephense-client",
        "onecentlin.laravel-blade",
        "bradlc.vscode-tailwindcss",
        "ms-vscode.vscode-typescript-next",
        "esbenp.prettier-vscode",
        "ms-vscode.vscode-json"
    ]
}
```

#### PHPStorm Configuration
- Install Laravel Plugin
- Configure PHP interpreter ke versi 8.1+
- Setup Composer dan NPM executables
- Configure database connection

### 4. Git Hooks (Opsional)

```bash
# Install pre-commit hooks
composer require --dev phpstan/phpstan
npm install --save-dev husky lint-staged

# Setup hooks
npx husky install
npx husky add .husky/pre-commit "npx lint-staged"
```

## Development Workflow

### 1. Git Workflow

#### Branch Naming Convention
```
feature/IKM-123-add-export-feature
bugfix/IKM-456-fix-calculation-error
hotfix/IKM-789-critical-security-fix
```

#### Commit Message Format
```
type(scope): subject

body (optional)

footer (optional)

# Examples:
feat(survey): add multi-language support
fix(calculation): correct IKM formula implementation
docs(readme): update installation instructions
```

#### Development Process
1. **Create Feature Branch**
   ```bash
   git checkout -b feature/IKM-123-new-feature
   ```

2. **Development**
   - Write code following coding standards
   - Write tests for new functionality
   - Update documentation if needed

3. **Before Commit**
   ```bash
   # Run linting
   ./vendor/bin/pint
   
   # Run tests
   php artisan test
   
   # Check static analysis
   ./vendor/bin/phpstan analyse
   ```

4. **Commit and Push**
   ```bash
   git add .
   git commit -m "feat(survey): add new feature"
   git push origin feature/IKM-123-new-feature
   ```

5. **Pull Request**
   - Create PR dengan template yang telah disediakan
   - Pastikan semua CI checks passed
   - Request review dari team lead

### 2. Daily Development Tasks

#### Start Development Session
```bash
# Pull latest changes
git pull origin main

# Start services
php artisan serve &
npm run dev
```

#### Code Quality Checks
```bash
# Format code
./vendor/bin/pint

# Static analysis
./vendor/bin/phpstan analyse

# Run tests
php artisan test
```

## Coding Standards

### 1. PHP Standards (PSR-12)

#### Class Structure
```php
<?php

namespace App\Http\Controllers;

use App\Models\Survey;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class SurveyController extends Controller
{
    /**
     * Display a listing of surveys.
     */
    public function index(Request $request): JsonResponse
    {
        $surveys = Survey::query()
            ->when($request->filled('search'), function ($query) use ($request) {
                $query->where('title', 'like', "%{$request->search}%");
            })
            ->paginate(15);

        return response()->json($surveys);
    }
}
```

#### Model Standards
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Kuesioner extends Model
{
    use HasFactory;

    protected $fillable = [
        'village_id',
        'unsur_id',
        'question',
        'is_active',
    ];

    protected $casts = [
        'is_active' => 'boolean',
    ];

    /**
     * Get the answers for the questionnaire.
     */
    public function answers(): HasMany
    {
        return $this->hasMany(Answer::class);
    }
}
```

### 2. JavaScript Standards

#### ES6+ Syntax
```javascript
// Use const/let instead of var
const apiUrl = '/api/surveys';
let currentPage = 1;

// Arrow functions
const fetchSurveys = async (page = 1) => {
    try {
        const response = await fetch(`${apiUrl}?page=${page}`);
        return await response.json();
    } catch (error) {
        console.error('Error fetching surveys:', error);
        throw error;
    }
};

// Destructuring
const { data, meta } = await fetchSurveys();
```

#### Event Handling
```javascript
// Use event delegation
document.addEventListener('DOMContentLoaded', () => {
    const surveyForm = document.getElementById('survey-form');
    
    surveyForm?.addEventListener('submit', async (event) => {
        event.preventDefault();
        await handleSurveySubmit(event.target);
    });
});
```

### 3. Blade Template Standards

#### Component Usage
```blade
{{-- Use components for reusable elements --}}
<x-layout.main>
    <x-page-header title="Survey Management" />
    
    <div class="container mx-auto px-4">
        <x-alert type="success" :show="session('success')">
            {{ session('success') }}
        </x-alert>
        
        <x-table :data="$surveys" :columns="$columns" />
    </div>
</x-layout.main>
```

#### Data Handling
```blade
{{-- Always escape data --}}
<h1>{{ $survey->title }}</h1>

{{-- Use when directive for conditionals --}}
@if($surveys->count() > 0)
    <div class="grid gap-4">
        @foreach($surveys as $survey)
            <x-survey-card :survey="$survey" />
        @endforeach
    </div>
@else
    <x-empty-state message="No surveys found" />
@endif
```

### 4. CSS Standards (Tailwind)

#### Utility-First Approach
```blade
{{-- Use Tailwind utilities --}}
<div class="bg-white shadow-lg rounded-lg p-6 mb-4">
    <h2 class="text-xl font-semibold text-gray-800 mb-2">
        Survey Title
    </h2>
    <p class="text-gray-600 text-sm">
        Survey description here...
    </p>
</div>
```

#### Custom Components
```css
/* resources/css/app.css */
@layer components {
    .btn-primary {
        @apply bg-blue-600 hover:bg-blue-700 text-white font-medium py-2 px-4 rounded-lg transition-colors;
    }
    
    .form-input {
        @apply border border-gray-300 rounded-md px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500;
    }
}
```

## Database Development

### 1. Migration Best Practices

#### Creating Migrations
```bash
# Create migration
php artisan make:migration create_surveys_table

# Migration with model
php artisan make:model Survey -m

# Add column to existing table
php artisan make:migration add_column_to_surveys_table --table=surveys
```

#### Migration Structure
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('surveys', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('description')->nullable();
            $table->boolean('is_active')->default(true);
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->timestamps();
            
            // Indexes
            $table->index(['is_active', 'created_at']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('surveys');
    }
};
```

### 2. Seeder Development

#### Model Factory
```php
<?php

namespace Database\Factories;

use App\Models\Village;
use Illuminate\Database\Eloquent\Factories\Factory;

class SurveyFactory extends Factory
{
    public function definition(): array
    {
        return [
            'title' => $this->faker->sentence(),
            'description' => $this->faker->paragraph(),
            'is_active' => $this->faker->boolean(80),
            'village_id' => Village::factory(),
        ];
    }
}
```

#### Seeder Class
```php
<?php

namespace Database\Seeders;

use App\Models\Survey;
use Illuminate\Database\Seeder;

class SurveySeeder extends Seeder
{
    public function run(): void
    {
        Survey::factory()
            ->count(50)
            ->create();
    }
}
```

### 3. Query Optimization

#### Eloquent Best Practices
```php
// Good: Use eager loading
$surveys = Survey::with(['village', 'responses'])
    ->where('is_active', true)
    ->paginate(15);

// Bad: N+1 queries
$surveys = Survey::where('is_active', true)->get();
foreach ($surveys as $survey) {
    echo $survey->village->name; // N+1 query problem
}

// Good: Use select for specific columns
$surveys = Survey::select(['id', 'title', 'village_id'])
    ->where('is_active', true)
    ->get();

// Good: Use chunk for large datasets
Survey::chunk(100, function ($surveys) {
    foreach ($surveys as $survey) {
        // Process survey
    }
});
```

## Testing

### 1. Setup Testing Environment

#### PHPUnit Configuration
```xml
<!-- phpunit.xml -->
<phpunit>
    <testsuites>
        <testsuite name="Feature">
            <directory suffix="Test.php">./tests/Feature</directory>
        </testsuite>
        <testsuite name="Unit">
            <directory suffix="Test.php">./tests/Unit</directory>
        </testsuite>
    </testsuites>
    <env name="APP_ENV" value="testing"/>
    <env name="DB_CONNECTION" value="sqlite"/>
    <env name="DB_DATABASE" value=":memory:"/>
</phpunit>
```

### 2. Writing Tests

#### Feature Tests
```php
<?php

namespace Tests\Feature;

use App\Models\User;
use App\Models\Survey;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class SurveyManagementTest extends TestCase
{
    use RefreshDatabase;

    public function test_admin_can_create_survey(): void
    {
        $admin = User::factory()->create(['role' => 'admin_utama']);
        
        $this->actingAs($admin)
            ->post('/dasbor/kuesioner', [
                'title' => 'Test Survey',
                'description' => 'Test Description',
                'is_active' => true,
            ])
            ->assertRedirect()
            ->assertSessionHas('success');

        $this->assertDatabaseHas('surveys', [
            'title' => 'Test Survey',
        ]);
    }
}
```

#### Unit Tests
```php
<?php

namespace Tests\Unit;

use App\Models\Survey;
use App\Models\Response;
use Tests\TestCase;

class IkmCalculationTest extends TestCase
{
    public function test_ikm_calculation_is_correct(): void
    {
        $survey = Survey::factory()->create();
        $responses = Response::factory()
            ->count(10)
            ->for($survey)
            ->create(['rating' => 4]);

        $ikm = calculateIKM($survey->responses);

        $this->assertEquals(100, $ikm['konversiIKM']);
    }
}
```

### 3. Testing Commands

```bash
# Run all tests
php artisan test

# Run specific test file
php artisan test tests/Feature/SurveyTest.php

# Run with coverage
php artisan test --coverage

# Run specific test method
php artisan test --filter test_admin_can_create_survey

# Parallel testing
php artisan test --parallel
```

## Debugging

### 1. Laravel Debugging Tools

#### Debug Bar (Development)
```bash
composer require barryvdh/laravel-debugbar --dev
```

#### Telescope (Advanced Debugging)
```bash
composer require laravel/telescope --dev
php artisan telescope:install
php artisan migrate
```

### 2. Debugging Techniques

#### Using dd() and dump()
```php
// Debug variables
dd($survey, $responses);

// Non-stopping debug
dump($query->toSql(), $query->getBindings());

// Debug in Blade
@dd($surveys)
@dump($currentUser)
```

#### Query Debugging
```php
// Enable query logging
DB::enableQueryLog();

// Your queries here
$surveys = Survey::where('is_active', true)->get();

// Get executed queries
dd(DB::getQueryLog());
```

#### Log Debugging
```php
// Log information
Log::info('Survey created', ['survey_id' => $survey->id]);

// Log with context
Log::error('Export failed', [
    'user_id' => auth()->id(),
    'error' => $exception->getMessage(),
    'trace' => $exception->getTraceAsString(),
]);
```

### 3. Browser Debugging

#### JavaScript Console
```javascript
// Debug AJAX requests
fetch('/api/surveys')
    .then(response => {
        console.log('Response:', response);
        return response.json();
    })
    .then(data => console.log('Data:', data))
    .catch(error => console.error('Error:', error));
```

#### Vue DevTools (if using Vue)
```bash
# Install Vue DevTools browser extension
# Enable Vue debugging in development
```

## Performance Optimization

### 1. Database Optimization

#### Indexing Strategy
```php
// In migration
$table->index(['is_active', 'created_at']);
$table->index(['village_id', 'is_active']);

// Composite indexes for common queries
Schema::table('surveys', function (Blueprint $table) {
    $table->index(['village_id', 'is_active', 'created_at']);
});
```

#### Query Optimization
```php
// Use select to limit columns
$surveys = Survey::select(['id', 'title', 'village_id'])
    ->with(['village:id,name'])
    ->where('is_active', true)
    ->get();

// Use chunks for large datasets
Survey::chunk(100, function ($surveys) {
    foreach ($surveys as $survey) {
        // Process survey
    }
});
```

### 2. Caching Strategies

#### Route Caching
```bash
# Cache routes (production only)
php artisan route:cache

# Clear route cache
php artisan route:clear
```

#### View Caching
```bash
# Cache compiled views
php artisan view:cache

# Clear view cache
php artisan view:clear
```

#### Application Caching
```php
// Cache expensive calculations
$ikm = Cache::remember("ikm_village_{$villageId}", 3600, function () use ($villageId) {
    return $this->calculateIKM($villageId);
});

// Cache with tags (if using Redis)
Cache::tags(['surveys', 'village_1'])->put('survey_data', $data, 3600);
```

### 3. Asset Optimization

#### Vite Optimization
```javascript
// vite.config.js
export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
    build: {
        rollupOptions: {
            output: {
                manualChunks: {
                    vendor: ['chart.js', 'tom-select'],
                },
            },
        },
    },
});
```

#### Image Optimization
```php
// Use Laravel's image intervention
use Intervention\Image\Facades\Image;

$image = Image::make($file)
    ->resize(800, 600, function ($constraint) {
        $constraint->aspectRatio();
        $constraint->upsize();
    })
    ->save($path);
```

## Deployment

### 1. Production Checklist

```bash
# Update dependencies
composer install --no-dev --optimize-autoloader

# Build assets
npm run build

# Cache configuration
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Run migrations
php artisan migrate --force

# Clear caches
php artisan cache:clear
php artisan queue:restart
```

### 2. Environment Configuration

#### Production .env
```env
APP_ENV=production
APP_DEBUG=false
LOG_LEVEL=error

# Database
DB_CONNECTION=mysql
DB_HOST=your-db-host
DB_DATABASE=your-db-name

# Cache
CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis

# Mail
MAIL_MAILER=smtp
# ... mail configuration
```

### 3. Server Configuration

#### Nginx Configuration
```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/survey-app/public;

    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

### 4. Monitoring

#### Application Monitoring
```php
// Add to AppServiceProvider
if ($this->app->environment('production')) {
    $this->app['log']->useFiles('/var/log/survey-app/laravel.log');
}
```

#### Health Check Endpoint
```php
// routes/web.php
Route::get('/health', function () {
    return response()->json([
        'status' => 'ok',
        'timestamp' => now(),
        'services' => [
            'database' => DB::connection()->getPdo() ? 'up' : 'down',
            'cache' => Cache::store()->getStore() ? 'up' : 'down',
        ]
    ]);
});
```

---

Dengan mengikuti panduan ini, Anda dapat mengembangkan aplikasi dengan standar yang konsisten dan maintainable. Selalu ingat untuk menjalankan tests sebelum melakukan commit dan deployment ke production.