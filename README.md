# PostgreSQL Docker Setup

A simple Docker Compose configuration for running PostgreSQL with persistent data storage and easy configuration.

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
