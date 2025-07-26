# PostgreSQL Docker Template

A production-ready PostgreSQL setup with pgAdmin for database management and automated backup solutions.

## Quick Start

1. Copy environment configuration:
   ```bash
   cp .env.example .env
   ```

2. Update credentials in `.env` for production use

3. Start services:
   ```bash
   docker-compose up -d
   ```

4. Access pgAdmin at http://localhost:5050
    - Email: admin@example.com (configurable via PGADMIN_EMAIL)
    - Password: your_pgadmin_password_here (configurable via PGADMIN_PASSWORD)
    - PostgreSQL server connection is automatically preconfigured with saved password

**Note**: Server configuration from `pgadmin/servers.json` is loaded only on first startup. If you need to reset server
configurations, remove the pgadmin volume: `docker compose down -v`

## Configuration

### Environment Variables

| Variable                | Default           | Description                  |
|-------------------------|-------------------|------------------------------|
| `POSTGRES_USER`         | postgres          | Database superuser           |
| `POSTGRES_PASSWORD`     | -                 | Database password (required) |
| `POSTGRES_DB`           | postgres          | Default database name        |
| `POSTGRES_PORT`         | 5432              | PostgreSQL port              |
| `PGADMIN_EMAIL`         | admin@example.com | pgAdmin login email          |
| `PGADMIN_PASSWORD`      | -                 | pgAdmin password (required)  |
| `PGADMIN_PORT`          | 5050              | pgAdmin web interface port   |
| `BACKUP_SCHEDULE`       | 0 2 * * *         | Cron schedule for backups    |
| `BACKUP_RETENTION_DAYS` | 7                 | Days to keep backups         |

## Common Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f postgres
docker-compose logs -f pgadmin

# Connect to PostgreSQL
docker exec -it postgres_db psql -U postgres -d postgres

# Create a new database
docker exec -it postgres_db createdb -U postgres new_database

# List databases
docker exec -it postgres_db psql -U postgres -l

# Execute SQL file
docker exec -i postgres_db psql -U postgres -d postgres < script.sql
```

## Backup and Restore

```bash
# Manual backup
docker exec postgres_db pg_dump -U postgres postgres > backup.sql

# Manual backup (custom format)
docker exec postgres_db pg_dump -U postgres -Fc postgres > backup.dump

# Restore from SQL
docker exec -i postgres_db psql -U postgres -d postgres < backup.sql

# Restore from custom format
docker exec postgres_db pg_restore -U postgres -d postgres backup.dump
```

## User Management

```bash
# Create user
docker exec -it postgres_db psql -U postgres -c "CREATE USER newuser WITH PASSWORD 'password';"

# Grant privileges
docker exec -it postgres_db psql -U postgres -c "GRANT ALL PRIVILEGES ON DATABASE postgres TO newuser;"

# List users
docker exec -it postgres_db psql -U postgres -c "\\du"
```

## Monitoring

```bash
# Check service status
docker-compose ps

# Check database health
docker exec postgres_db pg_isready -U postgres

# Monitor resource usage
docker stats postgres_db

# Check database size
docker exec postgres_db psql -U postgres -c "SELECT pg_size_pretty(pg_database_size('postgres'));"

# Check active connections
docker exec postgres_db psql -U postgres -c "SELECT count(*) FROM pg_stat_activity;"
```

## Troubleshooting

**PostgreSQL won't start:**

```bash
# Check logs
docker-compose logs postgres

# Reset data directory if corrupted
docker-compose down -v
docker-compose up -d
```

**Performance issues:**

```bash
# Check current settings
docker exec postgres_db psql -U postgres -c "SHOW ALL;"

# Monitor active queries
docker exec postgres_db psql -U postgres -c "SELECT * FROM pg_stat_activity;"
```

**Backup failures:**

```bash
# Test manual backup
docker exec postgres_db pg_dump -U postgres postgres > test-backup.sql
```

## Production Notes

1. Change default passwords in `.env`
2. Configure proper backup strategy
3. Set up SSL/TLS for secure connections
4. Tune PostgreSQL settings based on your hardware
5. Monitor resource usage and adjust limits

## Features

- PostgreSQL with pgAdmin web interface
- Automated backup system with retention policy
- Health checks and logging
- Performance tuning for development and production
- Common PostgreSQL extensions pre-installed
- Docker networking for service isolation

## Directory Structure

```
.
├── docker-compose.yml          # Main compose configuration
├── .env                        # Environment variables (local)
├── .env.example                # Environment template
├── init/                       # Database initialization scripts
│   └── 01-extensions.sql       # PostgreSQL extensions setup
├── backups/                    # Database backups
└── pgadmin/                    # pgAdmin configuration
    ├── servers.json            # Preconfigured server connections
    └── pgpass                  # Password file for automatic login
```