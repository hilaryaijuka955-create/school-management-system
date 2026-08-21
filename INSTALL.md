# Installation Guide

## Table of Contents
1. [Local Development Setup](#local-development-setup)
2. [Online Deployment](#online-deployment)
3. [PWA Installation](#pwa-installation)
4. [Troubleshooting](#troubleshooting)

---

## Local Development Setup

### Prerequisites

- **Python 3.8+** - [Download Python](https://www.python.org/downloads/)
- **PostgreSQL 12+** or **MySQL 8+** - Database server
- **Git** - Version control
- **Modern Browser** - Chrome, Firefox, Safari, or Edge

### Step 1: Clone the Repository

```bash
git clone https://github.com/hilaryaijuka955-create/school-management-system.git
cd school-management-system
```

### Step 2: Set Up Database

#### Option A: PostgreSQL

```bash
# Create database
createdb school_management_db

# Create user (optional)
psql -c "CREATE USER school_admin WITH PASSWORD 'school_password';"
psql -c "ALTER ROLE school_admin WITH CREATEDB;"
```

#### Option B: MySQL

```bash
# Login to MySQL
mysql -u root -p

# In MySQL prompt:
CREATE DATABASE school_management_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'school_admin'@'localhost' IDENTIFIED BY 'school_password';
GRANT ALL PRIVILEGES ON school_management_db.* TO 'school_admin'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Step 3: Set Up Backend

```bash
cd backend

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Linux/Mac:
source venv/bin/activate

# On Windows:
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Create .env file
cp .env.example .env

# Edit .env with your database credentials
# Example for PostgreSQL:
# DATABASE_URL=postgresql://school_admin:school_password@localhost:5432/school_management_db
```

### Step 4: Initialize Database

```bash
cd backend

# Run migrations (create tables)
python -c "from app import create_app, db; app = create_app(); app.app_context().push(); db.create_all(); print('Database initialized')"

# Load sample data
python scripts/seed_database.py
```

### Step 5: Start Backend Server

```bash
cd backend
python app.py
```

You should see:
```
 * Running on http://127.0.0.1:5000
 * Press CTRL+C to quit
```

### Step 6: Start Frontend (New Terminal)

```bash
cd frontend

# Python 3:
python -m http.server 8000

# Or use Node http-server if installed:
# npx http-server -p 8000

# Or use any other local server
```

### Step 7: Access the Application

Open your browser and go to:
```
http://localhost:8000
```

### Step 8: Login with Demo Account

- **Email**: `admin@school.local`
- **Password**: `Admin@123`

---

## Demo Accounts

All demo accounts have password: `Demo@123` (except admin which is `Admin@123`)

| Role | Email | Password | Initial Access |
|------|-------|----------|-----------------|
| Super Administrator | admin@school.local | Admin@123 | Full system access |
| Headteacher | headteacher@school.local | Demo@123 | Dashboard, reports |
| Teacher | teacher@school.local | Demo@123 | Attendance, marks |
| Bursar | bursar@school.local | Demo@123 | Fees, payments |
| Parent | parent@school.local | Demo@123 | Child records |
| Student | student@school.local | Demo@123 | Personal records |

---

## Online Deployment

### Prerequisites for Deployment

- Cloud hosting (AWS, Heroku, DigitalOcean, Google Cloud, etc.)
- Domain name (optional but recommended)
- SSL/TLS certificate (free from Let's Encrypt)
- PostgreSQL hosted database (AWS RDS, Heroku Postgres, etc.)

### Option A: Deploy to Heroku

#### Backend

```bash
# Install Heroku CLI
# https://devcenter.heroku.com/articles/heroku-cli

# Login to Heroku
heroku login

# Create new app
heroku create school-management-backend

# Add PostgreSQL addon
heroku addons:create heroku-postgresql:hobby-dev -a school-management-backend

# Set environment variables
heroku config:set SECRET_KEY="your-secret-key" -a school-management-backend
heroku config:set JWT_SECRET="your-jwt-secret" -a school-management-backend
heroku config:set FRONTEND_URL="https://your-frontend-domain.com" -a school-management-backend

# Deploy
git push heroku main

# Run migrations
heroku run python -c "from app import create_app, db; app = create_app(); app.app_context().push(); db.create_all()" -a school-management-backend

# Load demo data
heroku run python scripts/seed_database.py -a school-management-backend
```

#### Frontend

For static frontend hosting:

1. **Netlify**
   ```bash
   npm install -g netlify-cli
   netlify deploy --dir frontend
   ```

2. **Vercel**
   ```bash
   npm install -g vercel
   vercel --prod frontend
   ```

3. **GitHub Pages**
   - Push frontend folder to `gh-pages` branch
   - Enable GitHub Pages in repository settings

### Option B: Deploy to AWS

#### Backend (EC2 + RDS)

```bash
# 1. Create EC2 instance
# - Ubuntu 20.04 LTS
# - t2.micro (free tier)
# - Security group: Allow HTTP, HTTPS, SSH

# 2. Connect to instance
ssh -i your-key.pem ubuntu@your-instance-ip

# 3. Install dependencies
sudo apt update
sudo apt install python3 python3-pip python3-venv nginx

# 4. Clone repository
git clone https://github.com/hilaryaijuka955-create/school-management-system.git
cd school-management-system/backend

# 5. Set up Python environment
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# 6. Create .env file with RDS database URL
cp .env.example .env
# Edit .env with your RDS endpoint

# 7. Configure Nginx as reverse proxy
# (See nginx-config-example.conf)

# 8. Start application with Gunicorn
pip install gunicorn
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

#### Frontend (S3 + CloudFront)

```bash
# 1. Create S3 bucket
aws s3 mb s3://your-school-frontend

# 2. Upload frontend files
aws s3 sync frontend/ s3://your-school-frontend

# 3. Create CloudFront distribution
# (Use S3 bucket as origin)

# 4. Update API_URL in frontend config to point to backend
```

### Option C: Deploy to DigitalOcean App Platform

```bash
# 1. Install doctl CLI
# https://docs.digitalocean.com/reference/doctl/

# 2. Create app.yaml
# (See deployment/app.yaml)

# 3. Deploy
doctl apps create --spec app.yaml
```

### Production Configuration Checklist

- [ ] Use HTTPS/SSL certificate
- [ ] Set `DEBUG=False`
- [ ] Change `SECRET_KEY`
- [ ] Change `JWT_SECRET`
- [ ] Configure database backups
- [ ] Set up monitoring and logging
- [ ] Enable CORS for frontend domain only
- [ ] Configure rate limiting
- [ ] Set up email notifications
- [ ] Enable audit logging
- [ ] Configure automated backups
- [ ] Test offline synchronization
- [ ] Test mobile responsiveness

### Database Backup Strategy

```bash
# PostgreSQL automated backup
pg_dump school_management_db > backup_$(date +%Y%m%d_%H%M%S).sql

# MySQL automated backup
mysqldump -u school_admin -p school_management_db > backup_$(date +%Y%m%d_%H%M%S).sql

# Schedule with cron (daily at 2 AM)
# 0 2 * * * /path/to/backup.sh
```

---

## PWA Installation

### On Android

1. Open the application in **Chrome** or **Firefox**
2. Look for "Install app" banner (usually at bottom)
3. Tap "Install"
4. App appears on home screen
5. Open from home screen

**Manual Installation:**
1. Open in Chrome
2. Tap menu (⋮) → "Install app"
3. Tap "Install"

### On Windows/Mac

1. Open in **Chrome**
2. Click install icon (top right address bar)
3. Click "Install"
4. App opens in standalone window
5. Shortcut appears on desktop

### On iOS (iPad/iPhone)

1. Open in **Safari**
2. Tap Share button
3. Scroll and tap "Add to Home Screen"
4. Name the app
5. Tap "Add"

### Offline Usage

After installation:
- App works without internet
- Previously loaded data remains available
- New data syncs automatically when online
- Check status: Look for "ONLINE" / "OFFLINE" indicator

---

## Environment Variables

### Required Variables

```
DATABASE_URL         # Database connection string
SECRET_KEY          # Flask secret key (use strong random value)
JWT_SECRET          # JWT signing secret (use strong random value)
API_URL             # Backend API URL
FRONTEND_URL        # Frontend URL
```

### Optional Variables

```
DEBUG               # Set to False in production
MAIL_SERVER         # Email server for notifications
PAYMENT_PROVIDER    # Payment gateway integration
LOG_LEVEL           # DEBUG, INFO, WARNING, ERROR
```

---

## Troubleshooting

### Backend Issues

#### Port Already in Use
```bash
# Find process using port 5000
lsof -i :5000

# Kill process
kill -9 <PID>

# Or use different port
python app.py --port 5001
```

#### Database Connection Error
```
Error: could not connect to server
```

**Solution:**
- Check database is running
- Verify DATABASE_URL in .env
- Check username/password
- Verify port (PostgreSQL: 5432, MySQL: 3306)

#### Module Not Found
```bash
pip install -r requirements.txt
```

#### Migration Issues
```bash
# Reset database (delete all data)
python -c "from app import create_app, db; app = create_app(); app.app_context().push(); db.drop_all()"

# Recreate
python -c "from app import create_app, db; app = create_app(); app.app_context().push(); db.create_all()"
```

### Frontend Issues

#### CORS Errors
**Error**: `Access to XMLHttpRequest blocked by CORS policy`

**Solution:**
- Update backend `.env`: `FRONTEND_URL=http://localhost:8000`
- Restart backend
- Clear browser cache

#### Service Worker Not Registering
**Error**: Service worker fails to register

**Solution:**
1. Clear browser cache
2. Check manifest.json is valid
3. Service worker must be served over HTTPS (or localhost)
4. Check browser console for errors

#### IndexedDB Not Working
**Error**: Offline features not available

**Solution:**
1. Check browser supports IndexedDB
2. Check browser storage is not full
3. Check browser privacy settings allow IndexedDB
4. Try private/incognito mode

#### PWA Won't Install
**Error**: Install button doesn't appear

**Solution:**
1. Check manifest.json is valid
2. Ensure HTTPS (except localhost)
3. Must have service worker
4. Must have app icons (192x192, 512x512)

### Database Issues

#### Cannot Connect to Database
```bash
# Test PostgreSQL connection
psql -U school_admin -d school_management_db -h localhost

# Test MySQL connection
mysql -u school_admin -p school_management_db -h localhost
```

#### Large Data Import Issues
```bash
# For PostgreSQL
psql -U school_admin -d school_management_db < backup.sql

# For MySQL
mysql -u school_admin -p school_management_db < backup.sql
```

### Synchronization Issues

#### Sync Queue Not Clearing
1. Check internet connection
2. Verify backend is running
3. Check API_URL in frontend config
4. Review error logs
5. Manually retry: Sync Dashboard → Retry Failed

#### Duplicate Records After Sync
1. Check database constraints
2. Verify sync queue handling
3. Review audit logs
4. Contact support with affected records

---

## Performance Optimization

### Backend
```python
# Enable query caching
CACHE_ENABLED = True
CACHE_TIMEOUT = 300

# Enable database connection pooling
SQLALCHEMY_ENGINE_OPTIONS = {
    'poolsize': 10,
    'pool_recycle': 3600,
}

# Enable gzip compression
COMPRESS_ENABLED = True
```

### Frontend
```javascript
// Lazy load modules
// Optimize images
// Enable gzip
// Minimize CSS/JS
```

### Database
```sql
-- Add indexes for frequently queried columns
CREATE INDEX idx_student_class ON students(class_id);
CREATE INDEX idx_attendance_date ON attendance(attendance_date);
CREATE INDEX idx_marks_exam ON marks(exam_id);
```

---

## Getting Help

1. **Check logs**: `logs/app.log`
2. **Review errors**: Browser console (F12)
3. **Check database**: Connect directly to verify data
4. **Enable debug mode**: `DEBUG=True` in .env
5. **GitHub Issues**: Report bugs on GitHub

---

## Next Steps

1. ✅ Installation complete
2. Read [User Guide](./docs/USER_GUIDE.md)
3. Configure your school details in Settings
4. Create administrator user
5. Add teachers
6. Register students
7. Create classes
8. Set up timetable
9. Define fees structure
10. Begin using the system

---

**Need Help?** Check the documentation or open an issue on GitHub.

**Last Updated**: August 21, 2026
