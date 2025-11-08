<div align="center">
  <a href="https://gemsofindia.org/">
    <img src="https://gemsofindia.org/logo.png" alt="Gems of India" width="50" height="50">
    <h1 style="margin-bottom: 0">gemsofindia.org</h1>
   </a>
    <h2 style="margin-top: 0">Deployment Guide</h2>
    <p>This document provides step-by-step process for deploying the gemsofindia.org application.</p>
</div>

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Environment Variables](#environment-variables)
3. [Available Scripts](#available-scripts)
4. [Local Development](#local-development)
5. [Production Deployment](#production-deployment)
6. [Database Management](#database-management)
7. [Scaling](#scaling)
8. [Monitoring](#monitoring)
9. [Backup & Recovery](#backup--recovery)
10. [Troubleshooting](#troubleshooting)
11. [Help](#help)

## Prerequisites

- [Node.js](https://nodejs.org) 18+
- [PostgreSQL](https://www.postgresql.org) 14+
- [Docker](https://www.docker.com) & [Docker Compose](https://docs.docker.com/compose) (recommended)
- [pnpm](https://pnpm.io) package manager
- [Git](https://git-scm.com) for version control
- [Redis](https://redis.io) for caching and sessions

## Environment Variables

### Required Variables

```env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/gems_prod

# NextAuth
NEXTAUTH_SECRET=your-secret-key
NEXTAUTH_URL=http://your-domain.com

# OAuth Providers (optional but recommended)
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=

# File Uploads
UPLOADTHING_SECRET=
UPLOADTHING_APP_ID=
```

## Available Scripts

This project includes the following scripts, which can be run with `pnpm run <script_name>`:

- `dev`: Starts the development server using Next.js with Turbopack.
- `build`: Creates a production-ready build of the application.
- `start`: Starts the production server.
- `lint`: Lints the codebase using Next.js's built-in ESLint configuration.
- `db:generate`: Generates database migration files with Drizzle Kit.
- `db:migrate`: Applies generated migrations to the database.
- `db:push`: Pushes the schema directly to the database (useful for development).
- `db:seed`: Seeds the database with dummy data.
- `db:studio`: Opens Drizzle Studio, a GUI for your database.
- `categories`: A script for managing categories.
- `docker:up:dev`: Start development environment with Docker.
- `docker:down`: Stop Docker containers.
- `docker:logs`: View Docker logs.
- `docker:up:prod`: Start production environment with Docker.

## Local Development

### Development Workflow

**What happens automatically?**

- PostgreSQL 17 database created and configured
- Redis 7 for caching and sessions
- Database schema migrated
- Dummy data seeded (users, entities, reviews, etc.)
- Development server starts with hot reload

**Access:**

- App: http://localhost:3000
- Database: `localhost:5432` (user: `gems`, password: `gems_password`)
- Redis: `localhost:6379`

**Test Users (after seeding):**

| Role      | Email                      |
| --------- | -------------------------- |
| Admin     | `rajesh.kumar@example.com` |
| Moderator | `priya.sharma@example.com` |
| User      | `amit.patel@example.com`   |

### STEP-1) Clone the repository

```bash
git clone https://github.com/your-org/gems-of-india.git
cd gems-of-india
```

### STEP-2) Set up development environment variables

```bash
# Edit .env with your configuration
cp .env.example .env
```

### STEP-3). Start the development environment

1. **Setup**

   ```bash
   pnpm install
   cp .env.example .env # Update environment variables, if needed
   ```

2. **Database**

   ```bash
   pnpm db:generate  # Generate migrations
   pnpm db:push      # Apply migrations
   pnpm db:seed      # Seed test data
   ```

3. **Development**

   ```bash
   pnpm dev  # Start development server
   ```

4. **Testing**
   ```bash
   pnpm test  # Run tests
   ```

> ![INFO](https://img.shields.io/badge/INFO-blue)
>
> The application will be available at [`http://localhost:3000`](http://localhost:3000)

### Docker Setup

The easiest way to get started is with Docker. This will set up PostgreSQL 17, Redis 7, and the Next.js app with one command:

```bash
# Start everything (runs in background)
pnpm docker:up:dev

# View logs
pnpm docker:logs

# Stop everything
pnpm docker:down
```

## Production Deployment

### 1. Server Requirements

- Linux server (Ubuntu 22.04 LTS recommended)
- 2+ CPU cores
- 4GB+ RAM
- 20GB+ disk space

### 2. Deployment with Docker (Recommended)

> ![IMPORTANT](https://img.shields.io/badge/IMPORTANT-yellow)
>
> Complete docker documentation explained in _production deployment_ can be found in [**DOCKER.md**](./DOCKER.md).

#### 2.1. Set up production environment

```bash
# Create production .env
cp .env.production.example .env.production
nano .env.production # Change environment variables as per your production setup
```

#### 2.2. Start production services

```bash
docker-compose -f docker-compose.prod.yml up -d --build
```

### 3. Deployment without Docker

#### 3.1. Install dependencies

```bash
pnpm install --production
```

#### 3.2. Build the application

```bash
pnpm build
```

#### 3.3. Start the production server

```bash
pnpm start
```

## Database Management

### Set up the database to run migrations

```bash
# Using Docker
docker-compose exec app pnpm db:push

# Or locally
pnpm db:push
```

Open `.env` and update the variables with your own values. At a minimum, you'll need to provide the `DATABASE_URL` for your PostgreSQL database.

## Scaling

### Horizontal Scaling

1. Set up a load balancer (e.g., Nginx, AWS ALB)
2. Deploy multiple app instances
3. Use a managed database service like supabase, vercel, etc.
4. Enable database read replicas

### Caching

- Implement Redis for session storage
- Use Vercel/Next.js edge caching
- Enable CDN for static assets

## Monitoring

### Application Logs

```bash
# View logs
docker-compose logs -f app

# View database logs
docker-compose logs -f db
```

### Monitoring Tools

- Feature to be added in future releases.

## Backup & Recovery

### Create a backup using [pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html)

```bash
# Using pg_dump
pg_dump -U postgres gems_prod > backup_$(date +%Y%m%d).sql

# Using Docker
Docker-compose exec -T db pg_dump -U postgres gems_prod > backup_$(date +%Y%m%d).sql
```

### Create Backup Automation Using Script & Crontab

1. Ceate a scirpt as `backup.sh` in the root directory.
2. Give permission to the script, i.e. `chmod +x backup.sh`
3. Copy the boilerplate script below to crate a backup script and add it to the crontab with desired frequency of backup.

   ```bash
   #!/bin/bash

   # Configuration
   BACKUP_DIR="/var/backups/gems-of-india"
   TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
   DB_NAME="gems_prod"
   DB_USER="postgres"
   RETENTION_DAYS=7

   # Create backup directory if it doesn't exist
   mkdir -p "$BACKUP_DIR"

   # Set proper permissions
   chmod 700 "$BACKUP_DIR"

   # Log function
   log() {
       echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" >> "$BACKUP_DIR/backup.log"
   }

   # Create backup
   log "Starting database backup for $DB_NAME"
   BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_${TIMESTAMP}.sql"

   # Perform the backup
   if pg_dump -U "$DB_USER" -F p -b -v -f "$BACKUP_FILE" "$DB_NAME"; then
       # Compress the backup
       gzip -f "$BACKUP_FILE"
       log "Backup completed successfully: ${BACKUP_FILE}.gz"

       # Clean up old backups
       find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz" -type f -mtime +$RETENTION_DAYS -delete
       log "Cleaned up backups older than $RETENTION_DAYS days"

       # Set proper permissions on the backup file
       chmod 600 "${BACKUP_FILE}.gz"
   else
       log "ERROR: Backup failed for $DB_NAME"
       exit 1
   fi
   ```

### Automated Backup Schedule

> ![INFO](https://img.shields.io/badge/NOTE-yellow)
>
> Kindly check [crontab.guru](https://crontab.guru) for more information on cron syntax, use what you need to set up a cron job for daily backups:

```bash
# Edit crontab
crontab -e

# Add this line (runs daily at 2 AM)
0 2 * * * ./backup-script.sh
```

### Recovery Process

1. Stop the application
2. Restore the database:
   ```bash
   psql -U postgres gems_prod < backup_20231001.sql
   ```
3. Restart the application

## Troubleshooting

### Common Issues

#### Database Connection Issues

- Verify `DATABASE_URL` in `.env`
- Check if PostgreSQL is running
- Ensure the database user has correct permissions

#### Build Failures

- Clear Next.js cache: `rm -rf .next`
- Reinstall dependencies: `rm -rf node_modules && pnpm install`
- Check Node.js version: `node -v`

#### File Upload Issues

- Verify upload credentials
- Check disk space
- Ensure upload directory is writable

## Help

- Check the [GitHub Issues](https://github.com/varunmara/gems-of-india/issues)
- Join our [Discord Server](https://discord.gg/cVCYec2jFA)
