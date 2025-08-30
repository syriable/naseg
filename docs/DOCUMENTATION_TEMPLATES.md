# Documentation Templates & CI/CD Pipeline

## Complete Development Workflow for Laravel 12 Fiverr Marketplace

This document provides standardized documentation templates, CI/CD configuration, and development guidelines for the modular marketplace platform.

---

## Module Documentation Template

### Template: Module README.md

```markdown
# [Module Name] Module

## Overview

Brief description of the module's purpose and main functionality within the Fiverr-like marketplace platform.

**Module Type**: [Core/Feature/Integration]  
**Dependencies**: [List of required modules]  
**Package Version**: v1.0.0  
**Laravel Version**: 12.x  
**PHP Version**: 8.3+  

## Business Context

### What This Module Does
- [Primary business function]
- [Secondary functions]
- [Integration points]

### Key Features
- [ ] Feature 1
- [ ] Feature 2  
- [ ] Feature 3

## Architecture

### Directory Structure
```
Modules/[ModuleName]/
├── Config/
│   └── config.php
├── Console/
│   └── Commands/
├── Database/
│   ├── Migrations/
│   ├── Seeders/
│   └── Factories/
├── Entities/
│   └── Models/
├── Http/
│   ├── Controllers/
│   ├── Middleware/
│   ├── Requests/
│   └── Resources/
├── Providers/
│   └── [ModuleName]ServiceProvider.php
├── Resources/
│   ├── assets/
│   ├── lang/
│   └── views/
├── Routes/
│   ├── api.php
│   └── web.php
├── Services/
├── Tests/
│   ├── Feature/
│   └── Unit/
├── composer.json
└── module.json
```

## Packages Used

### Mandatory Package Integration

#### spatie/laravel-medialibrary
- **Purpose**: [Specific use in this module]
- **Implementation**: [How it's used]
- **Collections**: [List of media collections]
- **Configuration**: [Any module-specific config]

#### spatie/laravel-data
- **Purpose**: [Specific use in this module]
- **DTOs Created**: [List of Data classes]
- **Validation**: [Validation rules implemented]
- **Transformations**: [Data transformations]

#### spatie/laravel-permission
- **Purpose**: [Specific use in this module] 
- **Roles**: [Roles defined by this module]
- **Permissions**: [Permissions managed]
- **Guards**: [Any custom guards]

#### cknow/laravel-money
- **Purpose**: [Specific use in this module]
- **Models Using Money**: [List of models]
- **Calculations**: [Business calculations performed]
- **Currency Support**: [Supported currencies]

#### spatie/laravel-model-states
- **Purpose**: [Specific use in this module]
- **State Machines**: [List of state classes]
- **Transitions**: [Key transitions managed]
- **Automation**: [Automated state changes]

#### spatie/laravel-settings
- **Purpose**: [Specific use in this module]
- **Settings Classes**: [List of setting classes]
- **Configuration**: [Settings managed]
- **Caching**: [Cache strategy used]

## Database Schema

### Tables Created

#### [table_name]
```sql
CREATE TABLE [table_name] (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    -- [Column definitions with comments]
    created_at TIMESTAMP NULL DEFAULT NULL,
    updated_at TIMESTAMP NULL DEFAULT NULL
);
```

### Relationships
- **Belongs To**: [List relationships]
- **Has Many**: [List relationships]
- **Many To Many**: [List relationships]

### Indexes
```sql
-- Performance optimization indexes
CREATE INDEX idx_[name] ON [table] ([columns]);
```

## API Endpoints

### Public API Routes
```php
// GET /api/[module]/[resource]
// POST /api/[module]/[resource]  
// PUT /api/[module]/[resource]/{id}
// DELETE /api/[module]/[resource]/{id}
```

### Authentication Required Routes
```php
// Routes requiring auth:api middleware
```

### Permission Protected Routes  
```php
// Routes with specific permission requirements
```

## Events & Listeners

### Events Fired
- `[ModuleName]\Events\[EventName]`: [Description]

### Event Listeners
- `[ModuleName]\Listeners\[ListenerName]`: [Description]

## Data Transfer Objects (DTOs)

### [DTOName]Data
```php
class [DTOName]Data extends Data
{
    public function __construct(
        public readonly string $property,
        // ... other properties
    ) {}
}
```

**Usage Example**:
```php
$data = [DTOName]Data::from($request);
```

## Service Classes

### [ServiceName]Service
```php
class [ServiceName]Service
{
    public function primaryMethod(): ReturnType
    {
        // Implementation
    }
}
```

## Filament Resources

### [ResourceName]Resource
- **Purpose**: [Admin interface description]
- **Features**: [List of admin features]
- **Permissions**: [Required permissions]

## Livewire Components

### [ComponentName]
- **Purpose**: [Frontend component description]  
- **Features**: [Interactive features]
- **Dependencies**: [Required assets/scripts]

## Configuration

### Module Configuration (Config/config.php)
```php
return [
    'feature_flag' => env('MODULE_FEATURE_ENABLED', true),
    'settings' => [
        'key' => 'default_value',
    ],
];
```

### Environment Variables
```bash
# Module-specific environment variables
MODULE_SETTING=value
MODULE_API_KEY=secret
```

## Testing

### Test Coverage Goals
- **Unit Tests**: 90%+ coverage
- **Feature Tests**: All API endpoints
- **Integration Tests**: Cross-module interactions

### Running Tests
```bash
# Run all module tests
php artisan test Modules/[ModuleName]/Tests/

# Run specific test suite
php artisan test --filter [TestName]

# Run with coverage
php artisan test --coverage-min=90
```

### Test Examples

#### Unit Test Template
```php
<?php

namespace Modules\[ModuleName]\Tests\Unit;

use Tests\TestCase;
use Modules\[ModuleName]\Models\[ModelName];

class [ModelName]Test extends TestCase
{
    /** @test */
    public function it_can_create_[model](): void
    {
        // Arrange
        $data = [ModelName]::factory()->make()->toArray();
        
        // Act
        $model = [ModelName]::create($data);
        
        // Assert
        $this->assertDatabaseHas('[table_name]', $data);
    }
}
```

#### Feature Test Template  
```php
<?php

namespace Modules\[ModuleName]\Tests\Feature;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class [Feature]Test extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function [test_description](): void
    {
        // Arrange
        $user = User::factory()->create();
        
        // Act
        $response = $this->actingAs($user)
            ->postJson('/api/[endpoint]', []);
            
        // Assert
        $response->assertStatus(200);
    }
}
```

## Performance Considerations

### Database Optimization
- [List of optimizations implemented]
- [Indexing strategy]
- [Query optimization notes]

### Caching Strategy
- [Cache keys and TTL]
- [Cache invalidation rules]
- [Performance metrics]

## Security Considerations

### Input Validation
- [Validation rules applied]
- [Sanitization methods]
- [CSRF protection]

### Authorization
- [Permission checks]
- [Rate limiting]
- [Data access controls]

## Deployment Notes

### Migration Order
1. [Migration 1 description]
2. [Migration 2 description]

### Seeding Requirements
```bash
php artisan db:seed --class=[ModuleName]DatabaseSeeder
```

### Post-Deployment Steps
- [ ] Clear caches
- [ ] Reindex search
- [ ] Update permissions

## Monitoring & Logging

### Log Channels
- `[module-name]`: [Description of what gets logged]

### Metrics Tracked
- [Business metric 1]
- [Performance metric 1]

### Health Checks
```php
// Health check endpoint implementation
```

## Troubleshooting

### Common Issues

#### Issue: [Problem Description]
**Symptoms**: [How to identify]
**Solution**: [Step-by-step fix]
**Prevention**: [How to avoid]

## Contributing

### Development Setup
1. [Setup step 1]
2. [Setup step 2]

### Coding Standards
- Follow PSR-12 coding standards
- Use PHP 8.3 strict types  
- Comprehensive PHPDoc comments
- 100% type hints on public methods

### Git Workflow
```bash
# Feature branch naming
feature/[module-name]/[feature-description]

# Commit message format
[ModuleName]: Brief description

Detailed description if needed
```

## Changelog

### v1.0.0 (YYYY-MM-DD)
- Initial module implementation
- [Feature additions]
- [Bug fixes]

---

**Maintainer**: Development Team  
**Last Updated**: [Date]  
**Review Schedule**: Monthly
```

---

## CI/CD Pipeline Configuration

### GitHub Actions Workflow (.github/workflows/ci.yml)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

env:
  PHP_VERSION: 8.3
  NODE_VERSION: 20
  MYSQL_VERSION: 8.0

jobs:
  # Code Quality & Security Checks
  code-quality:
    name: Code Quality & Security
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ env.PHP_VERSION }}
          extensions: mbstring, xml, ctype, iconv, intl, pdo, pdo_mysql, dom, filter, gd, iconv, json, mbstring, redis
          coverage: xdebug
          
      - name: Cache Composer packages
        id: composer-cache
        uses: actions/cache@v3
        with:
          path: vendor
          key: ${{ runner.os }}-php-${{ hashFiles('**/composer.lock') }}
          restore-keys: |
            ${{ runner.os }}-php-

      - name: Install Composer dependencies
        run: composer install --no-progress --prefer-dist --optimize-autoloader

      - name: Run PHP CS Fixer (dry run)
        run: vendor/bin/php-cs-fixer fix --dry-run --diff --config=.php-cs-fixer.php

      - name: Run PHPStan
        run: vendor/bin/phpstan analyse --memory-limit=2G

      - name: Run Rector (dry run)
        run: vendor/bin/rector process --dry-run

      - name: Security Audit
        run: composer audit

  # Package Integration Validation
  package-validation:
    name: Package Integration Validation
    runs-on: ubuntu-latest
    needs: code-quality
    
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: secret
          MYSQL_DATABASE: marketplace_test
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3

      redis:
        image: redis:7-alpine
        options: --health-cmd="redis-cli ping" --health-interval=10s --health-timeout=5s --health-retries=3

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ env.PHP_VERSION }}
          extensions: mbstring, xml, ctype, iconv, intl, pdo, pdo_mysql, dom, filter, gd, iconv, json, mbstring, redis
          coverage: xdebug

      - name: Install Composer dependencies
        run: composer install --no-progress --prefer-dist --optimize-autoloader

      - name: Setup Laravel environment
        run: |
          cp .env.ci .env
          php artisan key:generate
          php artisan config:cache

      - name: Run database migrations
        env:
          DB_CONNECTION: mysql
          DB_HOST: 127.0.0.1
          DB_PORT: 3306
          DB_DATABASE: marketplace_test
          DB_USERNAME: root
          DB_PASSWORD: secret
        run: |
          php artisan migrate:fresh --seed
          php artisan db:seed --class=TestDataSeeder

      - name: Validate Package Integration
        env:
          DB_CONNECTION: mysql
          DB_HOST: 127.0.0.1
          DB_PORT: 3306
          DB_DATABASE: marketplace_test
          DB_USERNAME: root
          DB_PASSWORD: secret
        run: |
          # Test spatie/laravel-medialibrary
          php artisan tinker --execute="
            \$user = App\Models\User::factory()->create();
            \$media = \$user->addMediaFromUrl('https://via.placeholder.com/640x480')->toMediaCollection('test');
            if (!\$media->exists()) throw new Exception('Media library integration failed');
            echo 'Media library: ✅' . PHP_EOL;
          "
          
          # Test spatie/laravel-permission
          php artisan tinker --execute="
            \$role = Spatie\Permission\Models\Role::create(['name' => 'test-role']);
            \$permission = Spatie\Permission\Models\Permission::create(['name' => 'test-permission']);
            \$role->givePermissionTo(\$permission);
            if (!\$role->hasPermissionTo('test-permission')) throw new Exception('Permission integration failed');
            echo 'Permissions: ✅' . PHP_EOL;
          "
          
          # Test cknow/laravel-money
          php artisan tinker --execute="
            \$money = Money\Money::USD(2500);
            if (\$money->getAmount() !== 2500) throw new Exception('Money integration failed');
            echo 'Money package: ✅' . PHP_EOL;
          "

          # Test spatie/laravel-data
          php artisan tinker --execute="
            use Tests\Fixtures\SimpleData;
            \$data = SimpleData::from(['name' => 'test', 'age' => 25]);
            if (\$data->name !== 'test') throw new Exception('Laravel Data integration failed');
            echo 'Laravel Data: ✅' . PHP_EOL;
          "

      - name: Test Module Loading
        run: |
          php artisan module:list
          php artisan module:seed --class=AuthDatabaseSeeder
          php artisan module:seed --class=GigsDatabaseSeeder

  # Comprehensive Testing Suite
  testing:
    name: Test Suite (PHP ${{ matrix.php }})
    runs-on: ubuntu-latest
    needs: package-validation
    
    strategy:
      fail-fast: false
      matrix:
        php: [8.3]
        laravel: [12.x]

    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: secret
          MYSQL_DATABASE: marketplace_test
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3

      redis:
        image: redis:7-alpine
        options: --health-cmd="redis-cli ping" --health-interval=10s --health-timeout=5s --health-retries=3

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php }}
          extensions: mbstring, xml, ctype, iconv, intl, pdo, pdo_mysql, dom, filter, gd, iconv, json, mbstring, redis
          coverage: xdebug

      - name: Install Composer dependencies
        run: composer install --no-progress --prefer-dist --optimize-autoloader

      - name: Setup Laravel environment
        run: |
          cp .env.ci .env
          php artisan key:generate

      - name: Run migrations
        env:
          DB_CONNECTION: mysql
          DB_HOST: 127.0.0.1
          DB_PORT: 3306
          DB_DATABASE: marketplace_test
          DB_USERNAME: root
          DB_PASSWORD: secret
        run: php artisan migrate:fresh --seed

      - name: Run PHPUnit tests
        env:
          DB_CONNECTION: mysql
          DB_HOST: 127.0.0.1
          DB_PORT: 3306
          DB_DATABASE: marketplace_test
          DB_USERNAME: root
          DB_PASSWORD: secret
          XDEBUG_MODE: coverage
        run: |
          vendor/bin/phpunit --coverage-clover=coverage.xml --coverage-html=coverage-report --log-junit=junit.xml

      - name: Run Pest tests
        env:
          DB_CONNECTION: mysql
          DB_HOST: 127.0.0.1
          DB_PORT: 3306
          DB_DATABASE: marketplace_test  
          DB_USERNAME: root
          DB_PASSWORD: secret
        run: |
          vendor/bin/pest --parallel --coverage --min=80

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
          flags: unittests
          name: codecov-umbrella

      - name: Test Module Specific Functionality
        run: |
          # Test each module independently
          php artisan test Modules/Auth/Tests/ --parallel
          php artisan test Modules/Gigs/Tests/ --parallel  
          php artisan test Modules/Orders/Tests/ --parallel
          php artisan test Modules/Payments/Tests/ --parallel

  # Frontend Build & Test
  frontend:
    name: Frontend Build & Test
    runs-on: ubuntu-latest
    needs: code-quality

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build assets
        run: npm run build

      - name: Run JavaScript tests
        run: npm test

      - name: Run TypeScript type checking
        run: npm run type-check

  # Security Scanning
  security:
    name: Security Scanning
    runs-on: ubuntu-latest
    needs: code-quality

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ env.PHP_VERSION }}

      - name: Install Composer dependencies
        run: composer install --no-dev --optimize-autoloader

      - name: Run security checker
        run: |
          # Check for known security vulnerabilities
          composer audit
          
          # Additional security checks can be added here
          # Example: PHP Security Checker, Psalm Security Analysis

      - name: Scan for secrets
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: main
          head: HEAD

  # Performance Testing
  performance:
    name: Performance Testing
    runs-on: ubuntu-latest
    needs: testing
    if: github.ref == 'refs/heads/main'

    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: secret
          MYSQL_DATABASE: marketplace_perf
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ env.PHP_VERSION }}
          extensions: mbstring, xml, ctype, iconv, intl, pdo, pdo_mysql, dom, filter, gd, iconv, json, mbstring

      - name: Install dependencies
        run: composer install --no-dev --optimize-autoloader

      - name: Setup Laravel
        run: |
          cp .env.performance .env
          php artisan key:generate
          php artisan config:cache
          php artisan route:cache
          php artisan view:cache

      - name: Run database setup
        env:
          DB_CONNECTION: mysql
          DB_HOST: 127.0.0.1
          DB_PORT: 3306
          DB_DATABASE: marketplace_perf
          DB_USERNAME: root
          DB_PASSWORD: secret
        run: |
          php artisan migrate:fresh
          php artisan db:seed --class=PerformanceTestSeeder

      - name: Performance benchmarks
        run: |
          # Add performance testing commands
          php artisan benchmark:api-endpoints
          php artisan benchmark:database-queries
          php artisan benchmark:gig-search

  # Deployment (only on main branch)
  deploy:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [testing, frontend, security, performance]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    environment:
      name: staging
      url: https://staging.marketplace.example.com

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to staging
        run: |
          echo "Deploying to staging environment..."
          # Add deployment script here
          # This could use Docker, Ansible, or other deployment tools

      - name: Run post-deployment tests
        run: |
          echo "Running post-deployment health checks..."
          # Add health check commands
          
      - name: Notify deployment
        run: |
          echo "Deployment successful! Notifying team..."
          # Add notification logic (Slack, email, etc.)
```

### Environment Configuration Files

#### .env.ci (CI Environment)
```bash
APP_NAME="Marketplace CI"
APP_ENV=testing
APP_KEY=base64:TEST_KEY_HERE
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=single
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=marketplace_test
DB_USERNAME=root
DB_PASSWORD=secret

BROADCAST_DRIVER=log
CACHE_DRIVER=redis
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=array

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=log

# Package-specific test settings
MEDIA_DISK=local
PERMISSION_CACHE_ENABLED=false
```

#### .env.performance (Performance Testing)
```bash
APP_NAME="Marketplace Performance"
APP_ENV=performance
APP_KEY=base64:PERFORMANCE_KEY_HERE
APP_DEBUG=false
APP_URL=http://localhost

LOG_CHANNEL=single
LOG_LEVEL=warning

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=marketplace_perf
DB_USERNAME=root
DB_PASSWORD=secret

CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

# Performance optimizations
OPCACHE_ENABLE=1
VIEW_CACHE=true
CONFIG_CACHE=true
ROUTE_CACHE=true
```

---

## PHP CS Fixer Configuration

### .php-cs-fixer.php

```php
<?php

declare(strict_types=1);

$finder = PhpCsFixer\Finder::create()
    ->in(__DIR__)
    ->exclude('bootstrap/cache')
    ->exclude('storage')
    ->exclude('vendor')
    ->exclude('node_modules')
    ->name('*.php')
    ->notName('*.blade.php')
    ->ignoreDotFiles(true)
    ->ignoreVCS(true);

return (new PhpCsFixer\Config())
    ->setRiskyAllowed(true)
    ->setRules([
        '@PSR12' => true,
        '@PHP83Migration' => true,
        'array_syntax' => ['syntax' => 'short'],
        'ordered_imports' => ['sort_algorithm' => 'alpha'],
        'no_unused_imports' => true,
        'not_operator_with_successor_space' => false,
        'trailing_comma_in_multiline' => true,
        'phpdoc_scalar' => true,
        'unary_operator_spaces' => true,
        'binary_operator_spaces' => true,
        'blank_line_before_statement' => [
            'statements' => ['break', 'continue', 'declare', 'return', 'throw', 'try'],
        ],
        'phpdoc_single_line_var_spacing' => true,
        'phpdoc_var_without_name' => true,
        'class_attributes_separation' => [
            'elements' => [
                'method' => 'one',
            ],
        ],
        'method_argument_space' => [
            'on_multiline' => 'ensure_fully_multiline',
            'keep_multiple_spaces_after_comma' => true,
        ],
        'single_trait_insert_per_statement' => true,
        'declare_strict_types' => true,
    ])
    ->setFinder($finder);
```

---

## PHPStan Configuration

### phpstan.neon

```neon
includes:
    - ./vendor/larastan/larastan/extension.neon
    - ./vendor/phpstan/phpstan-strict-rules/rules.neon

parameters:
    paths:
        - app
        - Modules
    level: 8
    ignoreErrors:
        - '#Unsafe usage of new static#'
    excludePaths:
        - */vendor/*
        - */storage/*
        - */bootstrap/cache/*
        - */node_modules/*
    checkMissingIterableValueType: false
    checkGenericClassInNonGenericObjectType: false
    reportUnmatchedIgnoredErrors: false
```

---

## Test Configuration

### phpunit.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="./vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true"
         processIsolation="false"
         stopOnFailure="false"
         cacheDirectory=".phpunit.cache"
         backupGlobals="false"
         backupStaticAttributes="false">
    <testsuites>
        <testsuite name="Unit">
            <directory suffix="Test.php">./tests/Unit</directory>
            <directory suffix="Test.php">./Modules/*/Tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory suffix="Test.php">./tests/Feature</directory>
            <directory suffix="Test.php">./Modules/*/Tests/Feature</directory>
        </testsuite>
        <testsuite name="Integration">
            <directory suffix="Test.php">./tests/Integration</directory>
            <directory suffix="Test.php">./Modules/*/Tests/Integration</directory>
        </testsuite>
    </testsuites>
    <coverage processUncoveredFiles="true">
        <include>
            <directory suffix=".php">./app</directory>
            <directory suffix=".php">./Modules/*/app</directory>
            <directory suffix=".php">./Modules/*/Http</directory>
            <directory suffix=".php">./Modules/*/Models</directory>
            <directory suffix=".php">./Modules/*/Services</directory>
        </include>
        <exclude>
            <directory suffix=".php">./vendor</directory>
            <directory>./tests</directory>
            <directory>./Modules/*/Tests</directory>
        </exclude>
    </coverage>
    <php>
        <server name="APP_ENV" value="testing"/>
        <server name="BCRYPT_ROUNDS" value="4"/>
        <server name="CACHE_DRIVER" value="array"/>
        <server name="DB_CONNECTION" value="sqlite"/>
        <server name="DB_DATABASE" value=":memory:"/>
        <server name="MAIL_MAILER" value="array"/>
        <server name="QUEUE_CONNECTION" value="sync"/>
        <server name="SESSION_DRIVER" value="array"/>
        <server name="TELESCOPE_ENABLED" value="false"/>
    </php>
</phpunit>
```

### Pest Configuration (pest.php)

```php
<?php

declare(strict_types=1);

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithFaker;

uses(Tests\TestCase::class, RefreshDatabase::class, WithFaker::class)
    ->in('Feature');

uses(Tests\TestCase::class)
    ->in('Unit');

uses(Tests\TestCase::class, RefreshDatabase::class)
    ->in('Modules/*/Tests/Feature');

uses(Tests\TestCase::class)
    ->in('Modules/*/Tests/Unit');

// Global test helpers
function actingAsUser(string $role = 'buyer'): Tests\TestCase
{
    $user = \Modules\Auth\Models\User::factory()->create(['user_type' => $role]);
    return test()->actingAs($user);
}

function actingAsSeller(): Tests\TestCase
{
    return actingAsUser('seller');
}

function actingAsAdmin(): Tests\TestCase
{
    $admin = \Modules\Auth\Models\User::factory()->create();
    $admin->assignRole('admin');
    return test()->actingAs($admin);
}

// Custom expectations
expect()->extend('toBeValidMoney', function () {
    return $this->toBeInstanceOf(\Money\Money::class);
});

expect()->extend('toHaveValidState', function () {
    return $this->toBeInstanceOf(\Spatie\ModelStates\State::class);
});
```

This comprehensive documentation system ensures:

1. **Standardized Documentation** across all modules
2. **Complete CI/CD Pipeline** with quality gates
3. **Package Integration Validation** in every build
4. **Performance Monitoring** for critical paths
5. **Security Scanning** at multiple levels
6. **Automated Testing** with high coverage requirements
7. **Code Quality Enforcement** through static analysis
8. **Deployment Automation** with health checks

The system supports the modular architecture and ensures all mandated packages are properly integrated and tested.