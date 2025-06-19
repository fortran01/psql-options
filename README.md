# PostgreSQL Docker Setup

A simple Docker Compose configuration for running PostgreSQL with persistent data storage and easy configuration.

- [PostgreSQL Docker Setup](#postgresql-docker-setup)
  - [Prerequisites](#prerequisites)
  - [Quick Start](#quick-start)
  - [Configuration](#configuration)
    - [Environment Variables](#environment-variables)
    - [Custom Configuration](#custom-configuration)
  - [Data Persistence](#data-persistence)
  - [pgAdmin4 Web Interface](#pgadmin4-web-interface)
    - [Accessing pgAdmin4](#accessing-pgadmin4)
    - [Connecting to PostgreSQL](#connecting-to-postgresql)
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
  - [Alternative: Neon Launchpad (Instant Cloud PostgreSQL)](#alternative-neon-launchpad-instant-cloud-postgresql)
    - [Method 1: Web Interface](#method-1-web-interface)
    - [Method 2: CLI Command](#method-2-cli-command)
    - [Using Your Neon Database](#using-your-neon-database)
    - [Claiming Your Database](#claiming-your-database)
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

4. **Access pgAdmin4 Web Interface**:
   - Open http://localhost:8080 in your browser
   - Login with email: `admin@example.com` and password: `admin`
   - Add a new server connection to `postgres:5432`

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
| `PGADMIN_DEFAULT_EMAIL` | `admin@example.com` | pgAdmin4 login email |
| `PGADMIN_DEFAULT_PASSWORD` | `admin` | pgAdmin4 login password |

### Custom Configuration

You can modify the `docker-compose.yml` file to:

- Change the exposed ports (PostgreSQL: 5432, pgAdmin4: 8080)
- Add additional PostgreSQL configuration
- Mount custom configuration files
- Customize pgAdmin4 settings and authentication

## Data Persistence

Data is automatically persisted using Docker volumes:

- **PostgreSQL Data**:
  - Volume: `postgres_data`
  - Mount Point: `/var/lib/postgresql/data`
- **pgAdmin4 Data**:
  - Volume: `pgadmin_data`
  - Mount Point: `/var/lib/pgadmin`

Your data and pgAdmin4 configuration will survive container restarts and updates.

## pgAdmin4 Web Interface

This setup includes pgAdmin4, a web-based PostgreSQL administration tool that provides a graphical interface for managing your databases.

### Accessing pgAdmin4

1. **Start the services**:
   ```bash
   docker-compose up -d
   ```

2. **Open pgAdmin4 in your browser**:
   - URL: http://localhost:8080
   - Email: `admin@example.com`
   - Password: `admin`

### Connecting to PostgreSQL

Once logged into pgAdmin4:

1. **Add New Server**:
   - Right-click "Servers" in the left panel
   - Select "Register" > "Server..."

2. **Configure Connection**:
   - **General Tab**:
     - Name: `PostgreSQL Docker` (or any name you prefer)
   - **Connection Tab**:
     - Host name/address: `postgres` (the service name from docker-compose)
     - Port: `5432`
     - Username: `postgres`
     - Password: `password`

3. **Save and Connect**: Click "Save" to establish the connection

You can now use pgAdmin4's graphical interface to:
- Browse databases and schemas
- Run SQL queries
- Create and modify tables
- View data
- Monitor database performance
- Import/export data

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

1. **Ports already in use**:
   - PostgreSQL: Change the port mapping in `docker-compose.yml` from `5432:5432` to `5433:5432`
   - pgAdmin4: Change the port mapping from `8080:80` to `8081:80`
   - Connect using: `psql -h localhost -p 5433 -U postgres -d myapp`
   - Access pgAdmin4 at: http://localhost:8081

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

## Alternative: Neon Launchpad (Instant Cloud PostgreSQL)

If you want to get started with PostgreSQL instantly without any local setup, you can use [Neon Launchpad](https://neon.com/blog/neon-launchpad) - a tool that provides instant, hosted PostgreSQL databases in seconds with no signup required.

### Method 1: Web Interface

The fastest way to get a PostgreSQL database:

1. **Go to [neon.new](https://neon.new)** in your browser
2. **Get instant database credentials** - you'll receive a connection string immediately
3. **Use it like any PostgreSQL database** - works with any PostgreSQL-compatible tool

### Method 2: CLI Command

For command-line users and automation:

```bash
# Get an instant PostgreSQL database
npx neondb
```

This will:
- Create a real, hosted PostgreSQL database in 2 seconds
- Generate a `.env` file with standard environment variables
- Provide both direct and pooled connection strings

Example output in your `.env` file:
```bash
# Claimable DB expires at: {{ date string }}
# Claim it now to your account: https://neon.new/database/{{hash}}
DATABASE_URL=postgresql://...
DATABASE_URL_POOLER=postgresql://...
```

You can also use it with options:
```bash
# Skip prompts and use defaults
npx neondb --yes

# Specify custom .env file location
npx neondb --env ./custom/.env

# Set custom database URL key
npx neondb --key DB_CONNECTION_STRING
```

### Using Your Neon Database

The connection strings work with any PostgreSQL-compatible framework or tool:

```bash
# Connect using psql
psql $DATABASE_URL

# Use with your application
# The DATABASE_URL environment variable is automatically set
```

You can also use it to seed your database with the existing schema:

```bash
# Apply your initialization scripts
psql $DATABASE_URL < init-scripts/01-create-schema.sql
```

### Claiming Your Database

Databases created with Neon Launchpad:
- **Last 72 hours by default** - perfect for development and testing
- **Can be claimed later** - visit the claim URL provided to link it to a Neon account
- **Zero downtime claiming** - your connection strings continue to work after claiming
- **No interruptions** - your application stays connected during the claim process

To claim your database:
1. Visit the claim URL provided in the CLI output or `.env` file
2. Create a Neon account or sign in
3. The database is instantly linked to your account with no downtime

This makes Neon Launchpad perfect for:
- **Quick prototyping** - get a database instantly
- **Development workflows** - no local setup required  
- **CI/CD pipelines** - fresh databases for each test run
- **Demos and tutorials** - share working databases instantly
- **AI agents and platforms** - programmatic database provisioning

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
