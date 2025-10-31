# Household Beneficiary and Tranche Payment Management System

A secure, enterprise-grade web application to manage household beneficiaries and track up to three payment tranches per household.

## 🏗️ Architecture

- **Backend**: Laravel 10 with PostgreSQL
- **Frontend**: React 18 + TypeScript + Ant Design
- **Authentication**: Laravel Sanctum with role-based access control
- **Processing**: Background job queues for CSV imports
- **Deployment**: Docker containers with CI/CD pipeline

## 🚀 Features

- **CSV Import**: Validate, deduplicate, and import beneficiary data
- **Tranche Management**: Track up to 3 payment tranches per household
- **Role-Based Access**: Admin and StandardUser roles
- **Analytics Dashboard**: KPIs, demographic charts, payment tracking
- **Search & Filter**: Advanced filtering by location, status, dates
- **Export**: CSV/Excel export of filtered data
- **Audit Trail**: Complete audit logging for all operations

## 📁 Project Structure

```
├── backend/          # Laravel API backend
├── frontend/         # React TypeScript frontend
├── docker/           # Docker configuration files
├── docs/             # Documentation
├── postman/          # API testing collection
└── deploy/           # Deployment scripts
```

## 🛠️ Quick Start

### Prerequisites
- Docker & Docker Compose
- Node.js 18+ (for local development)
- PHP 8.1+ (for local development)

### Development Setup

1. **Clone and setup**
   ```bash
   git clone <repository-url>
   cd drive
   docker-compose up -d
   ```

2. **Backend setup**
   ```bash
   cd backend
   composer install
   cp .env.example .env
   php artisan key:generate
   php artisan migrate --seed
   ```

3. **Frontend setup**
   ```bash
   cd frontend
   npm install
   npm start
   ```

4. **Access the application**
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:8000
   - Database: PostgreSQL on port 5432

## 📊 Database Schema

### Core Tables
- `households` - Main beneficiary records (nidhh as primary key)
- `tranches` - Payment tranche details (normalized)
- `users` - System users with roles
- `audit_logs` - Complete audit trail
- `import_logs` - CSV import tracking

## 🔐 Authentication & Authorization

- **Admin Role**: Full system access, user management, system configuration
- **StandardUser Role**: View and edit beneficiary data, limited access

## 📈 CSV Import Process

1. **Upload**: Select CSV file with validation
2. **Preview**: Review data and potential issues
3. **Validate**: Check formats, duplicates, required fields
4. **Process**: Background job with progress tracking
5. **Report**: Detailed success/error reporting

## 🧪 Testing

```bash
# Backend tests
cd backend
php artisan test

# Frontend tests
cd frontend
npm test
```

## 📚 Documentation

- [API Documentation](docs/API.md)
- [User Guide](docs/USER_GUIDE.md)
- [Deployment Guide](docs/DEPLOYMENT.md)
- [CSV Format Guide](docs/CSV_FORMAT.md)

## 🚀 Deployment

See [Deployment Guide](docs/DEPLOYMENT.md) for production deployment instructions.

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
