# Panduan Kontribusi

Terima kasih atas minat Anda untuk berkontribusi pada Aplikasi Survei Kepuasan Masyarakat (IKM)! Dokumen ini berisi panduan lengkap untuk berkontribusi pada proyek ini.

## Daftar Isi

- [Cara Berkontribusi](#cara-berkontribusi)
- [Setup Development Environment](#setup-development-environment)
- [Coding Standards](#coding-standards)
- [Git Workflow](#git-workflow)
- [Pull Request Process](#pull-request-process)
- [Testing Guidelines](#testing-guidelines)
- [Documentation Guidelines](#documentation-guidelines)
- [Community Guidelines](#community-guidelines)

## Cara Berkontribusi

Ada beberapa cara Anda dapat berkontribusi pada proyek ini:

### 🐛 Melaporkan Bug
- Cek dulu apakah bug sudah dilaporkan di [Issues](https://github.com/Chandra-INF-D/surver-kepuasan-diskominfo/issues)
- Jika belum, buat issue baru dengan template bug report
- Berikan informasi selengkap mungkin untuk membantu reproduksi bug

### ✨ Mengusulkan Fitur Baru
- Diskusikan ide fitur baru di [Discussions](https://github.com/Chandra-INF-D/surver-kepuasan-diskominfo/discussions)
- Buat issue dengan template feature request
- Jelaskan use case dan manfaat fitur tersebut

### 📖 Meningkatkan Dokumentasi
- Perbaiki typo atau informasi yang tidak akurat
- Tambahkan contoh atau penjelasan yang lebih jelas
- Terjemahkan dokumentasi ke bahasa lain

### 💻 Kontribusi Kode
- Perbaiki bug yang dilaporkan
- Implementasi fitur baru yang telah disetujui
- Optimasi performance atau refactoring

### 🧪 Testing
- Tambahkan test cases untuk fitur yang belum ter-cover
- Tingkatkan test coverage
- Test manual untuk memastikan quality

## Setup Development Environment

### 1. Prerequisites

Pastikan Anda telah menginstall:
- PHP 8.1+
- Composer
- Node.js 16+
- MySQL/MariaDB
- Git

### 2. Fork dan Clone Repository

```bash
# Fork repository di GitHub UI
# Clone fork Anda
git clone https://github.com/YOUR_USERNAME/surver-kepuasan-diskominfo.git
cd surver-kepuasan-diskominfo

# Tambahkan upstream remote
git remote add upstream https://github.com/Chandra-INF-D/surver-kepuasan-diskominfo.git

# Verifikasi remotes
git remote -v
```

### 3. Setup Local Environment

```bash
# Install PHP dependencies
composer install

# Install JavaScript dependencies
npm install

# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate

# Setup database (edit .env first)
php artisan migrate
php artisan db:seed

# Build assets
npm run dev
```

### 4. Verify Setup

```bash
# Run tests
php artisan test

# Start development server
php artisan serve

# Start asset watcher
npm run dev
```

## Coding Standards

### 1. PHP Standards

Kami mengikuti [PSR-12](https://www.php-fig.org/psr/psr-12/) coding standard.

#### Formatting
```bash
# Format code menggunakan Laravel Pint
./vendor/bin/pint

# Check formatting (CI will fail if not formatted)
./vendor/bin/pint --test
```

#### Static Analysis
```bash
# Run PHPStan for static analysis
./vendor/bin/phpstan analyse

# Fix simple issues automatically
./vendor/bin/phpstan analyse --level=5
```

#### Naming Conventions
```php
// Classes: PascalCase
class SurveyController extends Controller

// Methods: camelCase
public function createSurvey()

// Variables: camelCase
$surveyData = [];

// Constants: UPPER_SNAKE_CASE
const MAX_QUESTIONS_PER_SURVEY = 50;

// Database tables: snake_case plural
Schema::create('survey_responses', function (Blueprint $table) {

// Database columns: snake_case
$table->string('question_text');
$table->timestamp('created_at');
```

### 2. JavaScript Standards

#### ES6+ Features
```javascript
// Use const/let instead of var
const apiEndpoint = '/api/surveys';
let currentPage = 1;

// Arrow functions for short functions
const fetchSurveys = async () => {
    // Implementation
};

// Template literals for strings
const message = `Loading page ${currentPage}...`;

// Destructuring
const { data, meta } = response;
```

#### Code Organization
```javascript
// Group imports
import { Chart } from 'chart.js';
import { Modal } from 'bootstrap';

// Use meaningful names
const showSuccessMessage = (message) => {
    // Implementation
};

// Add JSDoc comments for complex functions
/**
 * Calculate IKM score based on responses
 * @param {Array} responses - Array of survey responses
 * @returns {number} IKM score (0-100)
 */
const calculateIKMScore = (responses) => {
    // Implementation
};
```

### 3. Blade Template Standards

```blade
{{-- Use components for reusable elements --}}
<x-layout.main>
    <x-slot name="title">{{ $title }}</x-slot>
    
    <div class="container mx-auto px-4">
        {{-- Always escape output --}}
        <h1 class="text-2xl font-bold">{{ $survey->title }}</h1>
        
        {{-- Use @isset for optional variables --}}
        @isset($description)
            <p class="text-gray-600">{{ $description }}</p>
        @endisset
        
        {{-- Use @forelse for loops that might be empty --}}
        @forelse($questions as $question)
            <x-question-card :question="$question" />
        @empty
            <x-empty-state message="No questions found" />
        @endforelse
    </div>
</x-layout.main>
```

### 4. CSS Standards (Tailwind)

```css
/* Use Tailwind utilities first */
.survey-card {
    @apply bg-white rounded-lg shadow-md p-6 mb-4;
}

/* Custom CSS only when necessary */
.custom-gradient {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

/* Use meaningful class names */
.question-required::after {
    content: " *";
    @apply text-red-500;
}
```

## Git Workflow

### 1. Branch Naming Convention

```bash
# Feature branches
feature/IKM-123-add-export-functionality
feature/improve-dashboard-ui

# Bug fix branches
bugfix/IKM-456-fix-calculation-error
bugfix/resolve-login-redirect-issue

# Hotfix branches (for production issues)
hotfix/IKM-789-critical-security-fix

# Documentation branches
docs/update-installation-guide
docs/add-api-documentation
```

### 2. Commit Message Format

Gunakan [Conventional Commits](https://www.conventionalcommits.org/) format:

```
type(scope): subject

body (optional)

footer (optional)
```

#### Types
- `feat`: Fitur baru
- `fix`: Bug fix
- `docs`: Perubahan dokumentasi
- `style`: Perubahan formatting (tidak mengubah logic)
- `refactor`: Refactoring kode
- `test`: Menambah atau memperbaiki tests
- `chore`: Maintenance tasks

#### Examples
```bash
feat(survey): add multi-language support for questionnaires

fix(calculation): correct IKM formula implementation
- Fixed division by zero error
- Updated test cases to match new formula

docs(readme): update installation instructions

style(survey): format code according to PSR-12

refactor(controllers): extract common functionality to traits

test(ikm): add unit tests for calculation functions

chore(deps): update Laravel to version 10.x
```

### 3. Working with Branches

```bash
# Sync dengan upstream sebelum membuat branch baru
git checkout main
git pull upstream main

# Buat branch baru
git checkout -b feature/IKM-123-add-export

# Buat commits
git add .
git commit -m "feat(export): add basic export functionality"

# Push ke fork Anda
git push origin feature/IKM-123-add-export

# Setelah selesai, sync lagi dan rebase jika perlu
git checkout main
git pull upstream main
git checkout feature/IKM-123-add-export
git rebase main

# Push updated branch
git push origin feature/IKM-123-add-export --force-with-lease
```

## Pull Request Process

### 1. Sebelum Membuat PR

#### Checklist
- [ ] Code mengikuti coding standards
- [ ] Semua tests passing
- [ ] Dokumentasi sudah diupdate jika diperlukan
- [ ] Tidak ada merge conflicts
- [ ] Branch sudah di-rebase dengan main

#### Commands
```bash
# Run semua checks
./vendor/bin/pint
./vendor/bin/phpstan analyse
php artisan test
npm run build
```

### 2. Membuat Pull Request

#### PR Title Format
```
type(scope): short description

# Examples:
feat(survey): add export to Excel functionality
fix(auth): resolve OTP verification timeout issue
docs(api): add authentication endpoint documentation
```

#### PR Description Template
```markdown
## Description
Brief description of what this PR does.

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed

## Screenshots (if applicable)
Add screenshots for UI changes.

## Checklist
- [ ] My code follows the project's coding standards
- [ ] I have performed a self-review of my own code
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings
- [ ] Any dependent changes have been merged and published
```

### 3. Review Process

#### Untuk Reviewers
- Review code quality dan adherence ke standards
- Test functionality secara manual jika diperlukan
- Berikan feedback yang konstruktif
- Approve jika semua criteria terpenuhi

#### Untuk Contributors
- Respond terhadap feedback dengan profesional
- Make requested changes promptly
- Update PR dengan commits baru atau amend existing commits
- Resolve conversations setelah changes dibuat

### 4. Merge Guidelines

- **Squash and merge** untuk feature branches dengan multiple commits
- **Rebase and merge** untuk simple single-commit changes
- **Merge commit** untuk larger features yang ingin preserve commit history

## Testing Guidelines

### 1. Test Types

#### Unit Tests
```php
<?php

namespace Tests\Unit;

use App\Models\Survey;
use Tests\TestCase;

class IkmCalculationTest extends TestCase
{
    public function test_ikm_calculation_with_valid_responses()
    {
        // Arrange
        $responses = collect([
            ['answer' => 4],
            ['answer' => 3],
            ['answer' => 4],
        ]);

        // Act
        $result = calculateIKM($responses);

        // Assert
        $this->assertEquals(91.67, $result['konversiIKM'], '', 0.01);
    }

    public function test_ikm_calculation_with_empty_responses()
    {
        // Arrange
        $responses = collect([]);

        // Act & Assert
        $this->expectException(InvalidArgumentException::class);
        calculateIKM($responses);
    }
}
```

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

    public function test_admin_can_create_survey()
    {
        // Arrange
        $admin = User::factory()->create(['role' => 'admin_utama']);
        $surveyData = [
            'title' => 'Test Survey',
            'description' => 'Test Description',
            'is_active' => true,
        ];

        // Act
        $response = $this->actingAs($admin)
            ->post('/dasbor/kuesioner', $surveyData);

        // Assert
        $response->assertRedirect();
        $this->assertDatabaseHas('surveys', ['title' => 'Test Survey']);
    }

    public function test_guest_cannot_access_admin_panel()
    {
        // Act
        $response = $this->get('/dasbor');

        // Assert
        $response->assertRedirect('/login');
    }
}
```

### 2. Test Coverage

Target minimal test coverage adalah 80%.

```bash
# Generate coverage report
php artisan test --coverage

# Generate HTML coverage report
php artisan test --coverage-html coverage-report
```

### 3. Database Testing

```php
// Use factories untuk test data
$survey = Survey::factory()->create([
    'title' => 'Specific Title',
    'is_active' => true,
]);

// Use RefreshDatabase trait
use Illuminate\Foundation\Testing\RefreshDatabase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;
    
    // Tests here will have fresh database
}
```

## Documentation Guidelines

### 1. Code Documentation

#### PHP DocBlocks
```php
/**
 * Calculate the Community Satisfaction Index (IKM) based on survey responses.
 *
 * @param \Illuminate\Support\Collection $responses Collection of survey responses
 * @param \Illuminate\Support\Collection $questions Collection of survey questions
 * @return array{data: array, IKM: float, konversiIKM: float, bobotNilaiTertimbang: float}
 * @throws \InvalidArgumentException When responses collection is empty
 * 
 * @example
 * $result = getIKM($responses, $questions);
 * echo $result['konversiIKM']; // Outputs: 85.5
 */
function getIKM(Collection $responses, Collection $questions): array
{
    // Implementation
}
```

#### JavaScript JSDoc
```javascript
/**
 * Fetch survey data from the API
 * @param {number} page - Page number to fetch
 * @param {Object} filters - Optional filters
 * @param {string} filters.search - Search term
 * @param {number} filters.villageId - Village ID filter
 * @returns {Promise<Object>} Survey data with pagination info
 * @throws {Error} When API request fails
 */
async function fetchSurveys(page = 1, filters = {}) {
    // Implementation
}
```

### 2. Markdown Documentation

#### File Structure
```
docs/
├── INSTALASI.md          # Installation guide
├── ARSITEKTUR.md         # Architecture overview
├── PENGEMBANGAN.md       # Development guide
├── API.md                # API documentation
├── DEPLOYMENT.md         # Deployment guide
├── KONTRIBUSI.md         # This file
└── assets/               # Documentation assets
    ├── images/
    └── diagrams/
```

#### Writing Style
- Gunakan bahasa Indonesia yang jelas dan konsisten
- Sertakan contoh kode yang praktis
- Gunakan heading yang hierarkis
- Tambahkan table of contents untuk dokumen panjang
- Include screenshots untuk UI changes

### 3. API Documentation

Update dokumentasi API di `docs/API.md` ketika:
- Menambah endpoint baru
- Mengubah request/response format
- Mengubah parameter atau validation rules

## Community Guidelines

### 1. Code of Conduct

- **Be respectful**: Treat semua orang dengan hormat dan profesional
- **Be inclusive**: Welcome kontribusi dari semua background
- **Be constructive**: Berikan feedback yang membangun dan solusi
- **Be patient**: Understand bahwa semua orang memiliki level experience yang berbeda

### 2. Communication

#### Issue Templates
Gunakan template yang disediakan untuk:
- Bug reports
- Feature requests
- Documentation improvements

#### Discussion Etiquette
- Search dulu sebelum membuat thread baru
- Use descriptive titles
- Provide context dan examples
- Follow up on responses

### 3. Getting Help

Jika Anda membutuhkan bantuan:

1. **Documentation**: Baca dokumentasi terlebih dahulu
2. **Search Issues**: Cek apakah pertanyaan sudah pernah ditanyakan
3. **Discussions**: Post di GitHub Discussions untuk general questions
4. **Issues**: Create issue untuk bug reports atau feature requests

#### Questions Format
```markdown
## Environment
- OS: Ubuntu 22.04
- PHP: 8.1.12
- Laravel: 10.x
- Browser: Chrome 118

## What I'm trying to do
Clear description of what you want to achieve.

## What I've tried
- Step 1
- Step 2
- Step 3

## Error message
```
Copy exact error message here
```

## Expected vs Actual behavior
Expected: Should show success message
Actual: Shows error "..."
```

### 4. Recognition

Kontributor akan diakui melalui:
- GitHub contributors list
- Credits dalam dokumentasi
- Special mentions dalam release notes
- Invitation untuk menjadi maintainer bagi kontributor regular

## Getting Started Checklist

Untuk kontributor baru:

- [ ] Fork repository
- [ ] Setup development environment
- [ ] Run semua tests untuk memastikan setup benar
- [ ] Baca coding standards dan guidelines
- [ ] Pilih issue dengan label "good first issue"
- [ ] Buat branch untuk changes Anda
- [ ] Make your changes
- [ ] Write tests untuk changes Anda
- [ ] Update documentation jika diperlukan
- [ ] Create pull request
- [ ] Respond terhadap review feedback

---

Terima kasih sudah membaca panduan kontribusi ini! Kami menantikan kontribusi Anda untuk membuat aplikasi ini lebih baik. Jika ada pertanyaan, jangan ragu untuk bertanya di [Discussions](https://github.com/Chandra-INF-D/surver-kepuasan-diskominfo/discussions).