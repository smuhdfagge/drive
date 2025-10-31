# Deployment Guide

This guide covers deploying the Household Beneficiary Management System to production environments.

## Prerequisites

- Docker & Docker Compose
- PostgreSQL 15+
- Redis 7+
- Node.js 18+
- PHP 8.2+
- Nginx (for production)

## Environment Setup

### 1. Clone Repository

```bash
git clone <repository-url>
cd beneficiary-management-system
```

### 2. Environment Configuration

#### Backend Configuration
```bash
cd backend
cp .env.example .env
```

Edit `.env` with production values:
```env
APP_NAME="Beneficiary Management System"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://your-domain.com

DB_CONNECTION=pgsql
DB_HOST=your-db-host
DB_PORT=5432
DB_DATABASE=beneficiary_management
DB_USERNAME=your-db-user
DB_PASSWORD=your-secure-password

REDIS_HOST=your-redis-host
REDIS_PASSWORD=your-redis-password

SANCTUM_STATEFUL_DOMAINS=your-domain.com
FRONTEND_URL=https://your-domain.com
```

#### Frontend Configuration
```bash
cd frontend
cp .env.example .env
```

Edit `.env` with production values:
```env
REACT_APP_API_URL=https://your-domain.com/api
REACT_APP_ENV=production
```

## Deployment Options

### Option 1: Docker Compose (Recommended)

#### 1. Production Docker Compose

Create `docker-compose.prod.yml`:
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: beneficiary_management
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    restart: unless-stopped

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    environment:
      - APP_ENV=production
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./docker/nginx/prod.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - backend
      - frontend
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

#### 2. Build and Deploy

```bash
# Build production images
docker-compose -f docker-compose.prod.yml build

# Start services
docker-compose -f docker-compose.prod.yml up -d

# Run migrations
docker-compose -f docker-compose.prod.yml exec backend php artisan migrate --force

# Seed initial data
docker-compose -f docker-compose.prod.yml exec backend php artisan db:seed --force
```

### Option 2: Manual Deployment

#### 1. Database Setup

```sql
-- Create database
CREATE DATABASE beneficiary_management;
CREATE USER beneficiary_user WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE beneficiary_management TO beneficiary_user;
```

#### 2. Backend Deployment

```bash
cd backend

# Install dependencies
composer install --no-dev --optimize-autoloader

# Generate application key
php artisan key:generate

# Cache configuration
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Run migrations
php artisan migrate --force

# Seed database
php artisan db:seed --force

# Set permissions
chmod -R 755 storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache
```

#### 3. Frontend Deployment

```bash
cd frontend

# Install dependencies
npm ci

# Build for production
npm run build

# Copy build files to web server
cp -r build/* /var/www/html/
```

#### 4. Web Server Configuration

##### Nginx Configuration

Create `/etc/nginx/sites-available/beneficiary-management`:

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/html;
    index index.html;

    # Frontend routes
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API routes
    location /api {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
}
```

Enable site:
```bash
ln -s /etc/nginx/sites-available/beneficiary-management /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

## SSL Configuration

### Using Let's Encrypt

```bash
# Install Certbot
apt install certbot python3-certbot-nginx

# Obtain SSL certificate
certbot --nginx -d your-domain.com

# Auto-renewal
crontab -e
# Add: 0 12 * * * /usr/bin/certbot renew --quiet
```

## Process Management

### Using Supervisor for Laravel Queue

Create `/etc/supervisor/conf.d/beneficiary-worker.conf`:

```ini
[program:beneficiary-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /path/to/backend/artisan queue:work redis --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=2
redirect_stderr=true
stdout_logfile=/var/log/beneficiary-worker.log
stopwaitsecs=3600
```

Start supervisor:
```bash
supervisorctl reread
supervisorctl update
supervisorctl start beneficiary-worker:*
```

## Monitoring & Logging

### Application Logs

```bash
# Laravel logs
tail -f backend/storage/logs/laravel.log

# Nginx logs
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# PostgreSQL logs
tail -f /var/log/postgresql/postgresql-15-main.log
```

### Health Checks

Create health check endpoints:

```bash
# Backend health check
curl https://your-domain.com/api/health

# Database connectivity
curl https://your-domain.com/api/health/database
```

## Backup Strategy

### Database Backup

```bash
#!/bin/bash
# backup-db.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/database"
DB_NAME="beneficiary_management"

mkdir -p $BACKUP_DIR

pg_dump -h localhost -U beneficiary_user $DB_NAME | gzip > $BACKUP_DIR/backup_$DATE.sql.gz

# Keep only last 30 days
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +30 -delete
```

### File Backup

```bash
#!/bin/bash
# backup-files.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/files"
APP_DIR="/var/www/beneficiary-management"

mkdir -p $BACKUP_DIR

tar -czf $BACKUP_DIR/files_$DATE.tar.gz \
    $APP_DIR/backend/storage \
    $APP_DIR/backend/.env \
    --exclude="$APP_DIR/backend/storage/logs/*"

# Keep only last 7 days
find $BACKUP_DIR -name "files_*.tar.gz" -mtime +7 -delete
```

## Security Considerations

### 1. Environment Variables
- Never commit `.env` files
- Use strong passwords
- Rotate secrets regularly

### 2. Database Security
- Use dedicated database user
- Enable SSL connections
- Regular security updates

### 3. Application Security
- Keep dependencies updated
- Enable CSRF protection
- Use HTTPS only
- Implement rate limiting

### 4. Server Security
- Configure firewall
- Disable unused services
- Regular security patches
- Monitor access logs

## Performance Optimization

### 1. Database Optimization
```sql
-- Create indexes for common queries
CREATE INDEX idx_households_state ON households(state);
CREATE INDEX idx_households_lga ON households(lga);
CREATE INDEX idx_households_tranche_status ON households(tranche_status);
CREATE INDEX idx_tranches_payment_date ON tranches(payment_date);
```

### 2. Redis Caching
```bash
# Configure Redis for caching
redis-cli CONFIG SET maxmemory 256mb
redis-cli CONFIG SET maxmemory-policy allkeys-lru
```

### 3. PHP Optimization
```ini
; php.ini optimizations
opcache.enable=1
opcache.memory_consumption=256
opcache.max_accelerated_files=20000
opcache.validate_timestamps=0
```

## Troubleshooting

### Common Issues

1. **Database Connection Failed**
   ```bash
   # Check database status
   systemctl status postgresql
   
   # Test connection
   psql -h localhost -U beneficiary_user -d beneficiary_management
   ```

2. **Queue Jobs Not Processing**
   ```bash
   # Check queue worker status
   supervisorctl status beneficiary-worker:*
   
   # Restart workers
   supervisorctl restart beneficiary-worker:*
   ```

3. **High Memory Usage**
   ```bash
   # Monitor memory usage
   free -h
   
   # Check PHP processes
   ps aux | grep php
   ```

### Log Analysis

```bash
# Check for errors in Laravel logs
grep -i error backend/storage/logs/laravel.log

# Monitor failed jobs
php artisan queue:failed

# Check database slow queries
grep "slow query" /var/log/postgresql/postgresql-15-main.log
```

## Maintenance

### Regular Tasks

1. **Daily**
   - Monitor application logs
   - Check disk space
   - Verify backups

2. **Weekly**
   - Update dependencies
   - Review security logs
   - Performance monitoring

3. **Monthly**
   - Security patches
   - Database maintenance
   - Backup testing

### Update Process

```bash
# 1. Backup current version
./backup-db.sh
./backup-files.sh

# 2. Pull latest changes
git pull origin main

# 3. Update backend
cd backend
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan config:cache

# 4. Update frontend
cd ../frontend
npm ci
npm run build

# 5. Restart services
supervisorctl restart beneficiary-worker:*
systemctl reload nginx
```

This deployment guide provides a comprehensive approach to deploying the Beneficiary Management System in production environments with proper security, monitoring, and maintenance procedures.

