# PostgreSQL Docker Setup

A simple Docker Compose configuration for running PostgreSQL with persistent data storage and easy configuration.

- [PostgreSQL Docker Setup](#postgresql-docker-setup)
  - [Prerequisites](#prerequisites)
  - [Quick Start](#quick-start)
  - [Configuration](#configuration)
    - [Environment Variables](#environment-variables)
    - [Custom Configuration](#custom-configuration)
  - [Data Persistence](#data-persistence)
  - [Initialization Scripts](#initialization-scripts)
    - [Included Schema](#included-schema)
  - [Commands](#commands)
    - [Start Services](#start-services)
    - [Stop Services](#stop-services)
    - [View Logs](#view-logs)
    - [Connect to Database](#connect-to-database)
    - [Backup Database](#backup-database)
    - [Restore Database](#restore-database)
    - [Reset Database (⚠️ Destructive)](#reset-database-️-destructive)
    - [Working with the Schema](#working-with-the-schema)
  - [Health Check](#health-check)
  - [Troubleshooting](#troubleshooting)
    - [Common Issues](#common-issues)
    - [Useful Commands](#useful-commands)
  - [Security Considerations](#security-considerations)
  - [Alternative: Local PostgreSQL Installation](#alternative-local-postgresql-installation)
    - [Install PostgreSQL with Homebrew](#install-postgresql-with-homebrew)
    - [Start and Stop PostgreSQL Service](#start-and-stop-postgresql-service)
    - [Basic PostgreSQL Commands](#basic-postgresql-commands)
    - [Using pgAdmin](#using-pgadmin)
    - [Default Configuration](#default-configuration)
  - [PostgreSQL Version](#postgresql-version)
  - [Resources](#resources)

## Prerequisites

- Docker
- Docker Compose

## Quick Start

1. **Clone or download this repository**

2. **Start PostgreSQL**:
   ```bash
   docker-compose up -d
   ```

3. **Connect to PostgreSQL**:
   ```bash
   # Using Docker exec
   docker-compose exec postgres psql -U postgres -d myapp

   # Or using any PostgreSQL client
   psql -h localhost -p 5432 -U postgres -d myapp
   ```

## Configuration

### Environment Variables

Copy the example environment file and customize as needed:

```bash
cp .env.example .env
```

Available environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `POSTGRES_DB` | `myapp` | Default database name |
| `POSTGRES_USER` | `postgres` | PostgreSQL username |
| `POSTGRES_PASSWORD` | `password` | PostgreSQL password |

### Custom Configuration

You can modify the `docker-compose.yml` file to:

- Change the exposed port (default: 5432)
- Add additional PostgreSQL configuration
- Mount custom configuration files

## Data Persistence

Data is automatically persisted using Docker volumes:

- **Volume**: `postgres_data`
- **Mount Point**: `/var/lib/postgresql/data`

Your data will survive container restarts and updates.

## Initialization Scripts

Place SQL files in the `init-scripts/` directory to run them automatically when the database is first created:

```bash
mkdir init-scripts
echo "CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(100));" > init-scripts/01-create-tables.sql
```

Scripts are executed in alphabetical order.

### Included Schema

This setup includes a sample schema (`01-create-schema.sql`) that creates:

- **Schema**: `app` - A dedicated schema for application tables
- **Tables**:
  - `app.users` - User accounts with username, email, and profile info
  - `app.products` - Product catalog with pricing and inventory
  - `app.orders` - Customer orders with status tracking
  - `app.order_items` - Individual items within orders
- **Sample Data**: Pre-populated with example users and products
- **Indexes**: Optimized for common queries
- **View**: `app.order_summary` - Convenient order overview

To explore the schema:

```bash
# Connect to database
docker-compose exec postgres psql -U postgres -d myapp

# List all schemas
\dn

# Set search path to include app schema
SET search_path TO app, public;

# List tables in the app schema
\dt app.*

# View sample data
SELECT * FROM app.users;
SELECT * FROM app.products;
SELECT * FROM app.order_summary;
```

## Commands

### Start Services
```bash
docker-compose up -d
```

### Stop Services
```bash
docker-compose down
```

### View Logs
```bash
docker-compose logs -f postgres
```

### Connect to Database
```bash
# Interactive shell
docker-compose exec postgres psql -U postgres -d myapp

# Execute single command
docker-compose exec postgres psql -U postgres -d myapp -c "SELECT version();"
```

### Backup Database
```bash
docker-compose exec postgres pg_dump -U postgres myapp > backup.sql
```

### Restore Database
```bash
docker-compose exec -T postgres psql -U postgres myapp < backup.sql
```

### Reset Database (⚠️ Destructive)
```bash
docker-compose down -v  # Removes volumes and data
docker-compose up -d
```

### Working with the Schema
```bash
# Connect and set search path
docker-compose exec postgres psql -U postgres -d myapp -c "SET search_path TO app, public;"

# Query sample data
docker-compose exec postgres psql -U postgres -d myapp -c "SET search_path TO app, public; SELECT username, email FROM users;"

# View product catalog
docker-compose exec postgres psql -U postgres -d myapp -c "SET search_path TO app, public; SELECT name, price, category FROM products;"

# Check database structure
docker-compose exec postgres psql -U postgres -d myapp -c "\dt app.*"

# Create a sample order (interactive session recommended)
docker-compose exec postgres psql -U postgres -d myapp
```

Example queries to try in the interactive session:
```sql
-- Set search path
SET search_path TO app, public;

-- Create a sample order
INSERT INTO orders (user_id, total_amount, status) 
VALUES (1, 1029.98, 'pending');

-- Add items to the order
INSERT INTO order_items (order_id, product_id, quantity, unit_price) 
VALUES 
    (1, 1, 1, 999.99),  -- Laptop
    (1, 2, 1, 29.99);   -- Wireless Mouse

-- View the order summary
SELECT * FROM order_summary WHERE order_id = 1;
```

## Health Check

The container includes a health check that verifies PostgreSQL is accepting connections:

```bash
# Check container health
docker-compose ps
```

## Troubleshooting

### Common Issues

1. **Port already in use**:
   - Change the port mapping in `docker-compose.yml` from `5432:5432` to `5433:5432`
   - Connect using: `psql -h localhost -p 5433 -U postgres -d myapp`

2. **Permission denied errors**:
   - Ensure Docker has proper permissions
   - On Linux, you may need to run with `sudo`

3. **Data not persisting**:
   - Check that the volume is properly mounted
   - Verify with: `docker volume ls`

4. **Connection refused**:
   - Wait for the container to fully start (check logs)
   - Verify the container is running: `docker-compose ps`

### Useful Commands

```bash
# Check container status
docker-compose ps

# View detailed logs
docker-compose logs postgres

# Access container shell
docker-compose exec postgres bash

# Check PostgreSQL processes
docker-compose exec postgres ps aux

# Monitor resource usage
docker stats
```

## Security Considerations

For production use, consider:

1. **Change default passwords**: Use strong, unique passwords
2. **Limit network access**: Configure firewall rules
3. **Use secrets**: Store sensitive data in Docker secrets
4. **Regular updates**: Keep PostgreSQL image updated
5. **Backup strategy**: Implement automated backups

## Alternative: Local PostgreSQL Installation

If you prefer to run PostgreSQL locally without Docker, you can install it using Homebrew:

### Install PostgreSQL with Homebrew

```bash
# Install PostgreSQL
brew install postgresql@15

# Or install the latest version
brew install postgresql

# Install pgAdmin (PostgreSQL administration tool)
brew install --cask pgadmin4
```

### Start and Stop PostgreSQL Service

```bash
# Start PostgreSQL service
brew services start postgresql@15

# Stop PostgreSQL service
brew services stop postgresql@15

# Restart PostgreSQL service
brew services restart postgresql@15

# Check service status
brew services list | grep postgresql
```

### Basic PostgreSQL Commands

```bash
# Create a database
createdb myapp

# Connect to database
psql myapp

# Connect as specific user
psql -U postgres myapp

# List all databases
psql -l

# Drop a database
dropdb myapp
```

### Using pgAdmin

pgAdmin is a web-based PostgreSQL administration tool that provides a graphical interface for managing databases.

```bash
# Launch pgAdmin
open -a pgAdmin\ 4

# Or find it in Applications folder
# Applications > pgAdmin 4
```

**Setting up pgAdmin connection:**

1. Open pgAdmin (it will launch in your web browser)
2. Click "Add New Server"
3. Configure the connection:
   - **Name**: Local PostgreSQL (or any name you prefer)
   - **Host**: localhost
   - **Port**: 5432
   - **Username**: Your system username (or postgres if configured)
   - **Password**: Leave blank if no password is set, or enter your password

### Default Configuration

- **Default Port**: 5432
- **Default User**: Your system username
- **Data Directory**: `/opt/homebrew/var/postgresql@15/` (Apple Silicon) or `/usr/local/var/postgresql@15/` (Intel)
- **Config File**: `/opt/homebrew/etc/postgresql@15/postgresql.conf`

## PostgreSQL Version

This setup uses `postgres:latest`, which currently points to PostgreSQL 16. To use a specific version:

```yaml
services:
  postgres:
    image: postgres:15  # or postgres:14, postgres:13, etc.
```

## Resources

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [PostgreSQL Docker Hub](https://hub.docker.com/_/postgres)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
