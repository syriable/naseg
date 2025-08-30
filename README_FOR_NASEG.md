# Naseg - Fiverr-Like Marketplace

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PHP Version](https://img.shields.io/badge/PHP-8.3+-blue.svg)](https://www.php.net)
[![Laravel Version](https://img.shields.io/badge/Laravel-12.x-red.svg)](https://laravel.com)

A production-ready, enterprise-grade Fiverr-like marketplace built with Laravel 12, featuring modular architecture, comprehensive security, and full package integration.

## 🚀 Features

- **Modular Architecture** using nWidart/laravel-modules
- **Enterprise Security** with comprehensive risk assessment
- **Advanced Admin Interface** powered by Filament v4
- **Interactive Frontend** with Livewire v3 components
- **Complete Package Integration** with all Spatie packages
- **Financial Processing** with precise money handling
- **State Management** for order lifecycle
- **GDPR Compliance** built-in
- **Comprehensive Testing** framework

## 🏗️ Architecture

### Core Modules
- **Auth** - User authentication & profile management
- **Gigs** - Service creation & management
- **Orders** - Order processing with state machine
- **Payments** - Payment processing & escrow system
- **Reviews** - Rating & review system
- **Messaging** - In-platform communication
- **Disputes** - Resolution workflow
- **Analytics** - Business intelligence
- **Security** - Fraud detection & moderation

### Technology Stack
- **Framework**: Laravel 12.x
- **PHP**: 8.3+ with strict types
- **Frontend**: Livewire 3 + Tailwind CSS + Alpine.js
- **Admin**: Filament v4
- **Database**: MySQL 8.0+
- **Cache**: Redis
- **Queue**: Redis/Database
- **Storage**: Local/S3 compatible

## 📦 Package Integration

| Package | Purpose | Status |
|---------|---------|--------|
| spatie/laravel-medialibrary | Media management | ✅ Integrated |
| spatie/laravel-data | Type-safe DTOs | ✅ Integrated |
| spatie/laravel-settings | Configuration | ✅ Integrated |
| spatie/laravel-model-states | State machines | ✅ Integrated |
| spatie/laravel-permission | Access control | ✅ Integrated |
| cknow/laravel-money | Financial calculations | ✅ Integrated |
| nwidart/laravel-modules | Modular structure | ✅ Integrated |

## 📚 Documentation

- **[Complete Blueprint](docs/FIVERR_MARKETPLACE_BLUEPRINT.md)** - Full architecture and implementation guide
- **[Filament Resources](docs/FILAMENT_RESOURCES_EXAMPLES.md)** - Admin interface examples
- **[Livewire Components](docs/LIVEWIRE_COMPONENTS_EXAMPLES.md)** - Interactive frontend components
- **[Documentation Templates](docs/DOCUMENTATION_TEMPLATES.md)** - Development standards and CI/CD
- **[Security & Risk](docs/RISK_REGISTER_SECURITY.md)** - Security framework and risk assessment
- **[Executive Summary](docs/EXECUTIVE_SUMMARY.md)** - Project overview and implementation roadmap

## 🛡️ Security

This project implements enterprise-grade security with:
- 16 identified security risks with mitigation strategies
- GDPR compliance framework
- PCI-DSS compliant payment processing
- Multi-factor authentication
- Advanced rate limiting and DDoS protection
- Comprehensive input validation and sanitization

## 🏃 Quick Start

### Prerequisites
- PHP 8.3+
- Composer 2.0+
- Node.js 18+
- MySQL 8.0+ or PostgreSQL 14+
- Redis 6.0+

### Installation
```bash
# Clone the repository
git clone git@github.com:syriable/naseg.git
cd naseg

# Copy environment file
cp .env.example .env

# Install PHP dependencies
composer install

# Install Node dependencies
npm install

# Generate application key
php artisan key:generate

# Run migrations
php artisan migrate --seed

# Build assets
npm run build

# Start development server
php artisan serve
```

## 🧪 Testing

```bash
# Run all tests
php artisan test

# Run with coverage
php artisan test --coverage

# Run specific test suite
php artisan test --testsuite=Feature
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🏆 Acknowledgments

- Built following Laravel best practices and conventions
- Implements Fiverr.com business model patterns
- Uses industry-standard security frameworks
- Follows enterprise architecture principles

---

**Repository**: https://github.com/syriable/naseg  
**Project Status**: Blueprint Complete - Implementation In Progress  
**Version**: 1.0.0-alpha  
**Last Updated**: 2025-08-30