# Docker Deployment Guide for OpenBound Server-Side Services

This guide explains how to deploy the OpenBound server-side components using Docker and Docker Compose.

## Overview

The OpenBound server infrastructure consists of four main services:

1. **Avatar API** - ASP.NET Core Web API for payment gateway integrations (PayPal, Stripe)
2. **Login Server** - Handles player authentication and account creation
3. **Lobby Server** - Manages player UID generation, game server registration, and server lists
4. **Game Server** - Manages game rooms, matches, and in-game synchronization

## Prerequisites

- Docker Engine 20.10 or later
- Docker Compose 2.0 or later
- .NET 10.0 SDK (for development)
- SQL Server database (included in docker-compose.yml)

## Quick Start

### 1. Build and Run All Services

```bash
# Build and start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down
```

### 2. Individual Service Management

Build a specific service:
```bash
docker-compose build avatar-api
docker-compose build login-server
docker-compose build lobby-server
docker-compose build game-server
```

Start a specific service:
```bash
docker-compose up -d avatar-api
docker-compose up -d login-server
docker-compose up -d lobby-server
docker-compose up -d game-server
```

View logs for a specific service:
```bash
docker-compose logs -f avatar-api
docker-compose logs -f login-server
```

## Configuration

### Database Configuration

The included SQL Server container uses the following default settings:
- **Host**: database (container name)
- **Port**: 1433
- **Username**: sa
- **Password**: YourStrong@Passw0rd (⚠️ **CHANGE THIS IN PRODUCTION!**)

To use an external database, remove the `database` service from `docker-compose.yml` and configure connection strings in your application configuration files.

### Server Configuration

Each server creates its own configuration files on first run in the `/app/Config` directory within the container.

To persist and customize these configurations:

1. Create a local config directory structure:
```bash
mkdir -p config/login-server
mkdir -p config/lobby-server
mkdir -p config/game-server
```

2. Uncomment the volume mounts in `docker-compose.yml`:
```yaml
volumes:
  - ./config/login-server:/app/Config
```

3. Run the service once to generate default configuration files, then customize them as needed.

### Avatar API Configuration

The Avatar API uses `appsettings.json` for configuration. To customize:

1. Create `appsettings.Production.json` in the `Avatar API` directory
2. Update payment gateway credentials and database connection strings
3. Rebuild the container

### Port Mapping

Default port mappings (host:container):
- Avatar API: 5000:80 (HTTP)
- Login Server: 8022:8022
- Lobby Server: 8023:8023
- Game Server: 8024:8024
- Database: 1433:1433

To change host ports, edit the `ports` section in `docker-compose.yml`.

## Network Architecture

All services communicate through the `openbound-network` bridge network. This allows:
- Services to reference each other by container name
- Isolated network from host system
- Easy service discovery

Service dependencies:
```
Avatar API → Database
Login Server → Database, Lobby Server
Lobby Server → Database
Game Server → Database, Lobby Server
```

## Production Deployment

### Security Considerations

1. **Change Default Passwords**: Update the database password in `docker-compose.yml`
2. **Use Environment Variables**: Store sensitive data in environment variables or secrets
3. **Enable HTTPS**: Configure SSL certificates for Avatar API
4. **Firewall Rules**: Restrict access to database and internal ports
5. **Update Connection Strings**: Configure servers to use the database service name

### Environment Variables

You can use a `.env` file to manage environment variables:

```env
# .env file
DB_PASSWORD=YourSecurePassword123!
AVATAR_API_PORT=5000
LOGIN_SERVER_PORT=8022
LOBBY_SERVER_PORT=8023
GAME_SERVER_PORT=8024
```

Then reference in `docker-compose.yml`:
```yaml
environment:
  - SA_PASSWORD=${DB_PASSWORD}
```

### Persistent Data

The database data is persisted using Docker volumes. To backup:
```bash
# Backup
docker run --rm -v openbound_mssql-data:/data -v $(pwd):/backup ubuntu tar czf /backup/db-backup.tar.gz /data

# Restore
docker run --rm -v openbound_mssql-data:/data -v $(pwd):/backup ubuntu tar xzf /backup/db-backup.tar.gz -C /
```

## Build Process

Each service uses multi-stage Docker builds:

1. **Build Stage**: Uses `mcr.microsoft.com/dotnet/sdk:10.0` to compile the application
2. **Publish Stage**: Creates optimized release binaries
3. **Runtime Stage**: Uses smaller runtime images (`aspnet:10.0` or `runtime:10.0`)

This approach:
- Minimizes final image size
- Improves security (no SDK tools in production)
- Speeds up deployments

## Troubleshooting

### Check Service Status
```bash
docker-compose ps
```

### View Detailed Logs
```bash
docker-compose logs -f [service-name]
```

### Rebuild After Code Changes
```bash
docker-compose build --no-cache [service-name]
docker-compose up -d [service-name]
```

### Database Connection Issues
```bash
# Check if database is healthy
docker-compose exec database /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P YourStrong@Passw0rd -Q "SELECT 1" -C

# View database logs
docker-compose logs database
```

### Network Issues
```bash
# Inspect network
docker network inspect openbound_openbound-network

# Test connectivity between services
docker-compose exec login-server ping lobby-server
```

## Development Workflow

For development, you can mount source code as volumes for live reloading:

```yaml
volumes:
  - ./Avatar API:/src/Avatar API
  - ./OpenBound Network Object Library:/src/OpenBound Network Object Library
```

## Scaling

To run multiple game servers:

```bash
docker-compose up -d --scale game-server=3
```

Note: You'll need to configure different ports for each instance or use a load balancer.

## Monitoring and Logs

Logs are configured with rotation:
- Maximum file size: 10MB
- Maximum files: 5
- Total log storage: ~50MB per service

To adjust, modify the `logging` section in `docker-compose.yml`.

## Container Management

### Stop and Remove Containers
```bash
docker-compose down
```

### Stop, Remove, and Delete Volumes
```bash
docker-compose down -v
```

### Prune Unused Resources
```bash
docker system prune -a
```

## Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [.NET Docker Images](https://hub.docker.com/_/microsoft-dotnet)
- [SQL Server on Docker](https://hub.docker.com/_/microsoft-mssql-server)

## Support

For issues and questions, please refer to the main project README or open an issue on the project repository.
