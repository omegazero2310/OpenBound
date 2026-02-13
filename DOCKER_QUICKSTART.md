# Docker Quick Start Guide

## Prerequisites
- Docker Engine 20.10+
- Docker Compose 2.0+

## Quick Commands

### Start All Services
```bash
docker compose up -d
```

### View Logs
```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f avatar-api
docker compose logs -f login-server
docker compose logs -f lobby-server
docker compose logs -f game-server
```

### Stop All Services
```bash
docker compose down
```

### Check Service Status
```bash
docker compose ps
```

### Rebuild After Code Changes
```bash
# Rebuild all
docker compose build

# Rebuild specific service
docker compose build avatar-api
docker compose up -d avatar-api
```

## Service URLs
- **Avatar API**: http://localhost:5000
- **Avatar API Swagger**: http://localhost:5000/swagger
- **Login Server**: localhost:8022 (TCP)
- **Lobby Server**: localhost:8023 (TCP)
- **Game Server**: localhost:8024 (TCP)
- **Database**: localhost:1433 (SQL Server)

## Default Database Credentials
- **Host**: localhost (or `database` from within containers)
- **Port**: 1433
- **Username**: sa
- **Password**: YourStrong@Passw0rd

⚠️ **IMPORTANT**: Change the default password in production!

## Configuration
1. Copy `.env.example` to `.env`
2. Update values in `.env`
3. Restart services: `docker compose up -d`

## Troubleshooting

### Database connection issues
```bash
# Check database health
docker compose exec database /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P YourStrong@Passw0rd -Q "SELECT 1" -C
```

### Service won't start
```bash
# View detailed logs
docker compose logs [service-name]

# Restart specific service
docker compose restart [service-name]
```

### Clean rebuild
```bash
# Remove all containers and volumes
docker compose down -v

# Rebuild and start
docker compose up -d --build
```

## For More Information
See [DOCKER_DEPLOYMENT.md](DOCKER_DEPLOYMENT.md) for comprehensive documentation.
