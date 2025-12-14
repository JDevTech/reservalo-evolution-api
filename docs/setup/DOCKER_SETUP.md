# Evolution API - Docker Setup Guide

This guide will help you run Evolution API using Docker for local development.

## Prerequisites

- **Docker Desktop** installed and running
- At least 4GB of free RAM
- Ports available: `8080` (API), `3333` (Frontend), `5433` (PostgreSQL), `6379` (Redis)

## Quick Start

### Step 1: Verify Docker is Running

```bash
docker --version
docker-compose --version
```

If Docker is not running, start **Docker Desktop** application.

### Step 2: Navigate to Evolution API Directory

```bash
cd /Users/jd/Desktop/Projects/Cholao/Development/Reservalo/Backend/reservalo-evolution-api
```

### Step 3: Configuration (Already Done ✅)

The `.env` file has been created and configured with the following settings:

- **API Port**: 8080
- **Frontend Port**: 3333 (changed from 3000 to avoid conflict with Resérvalo Backend)
- **PostgreSQL Port**: 5433 (mapped to avoid conflict with Resérvalo Backend on 5432)
- **Redis Port**: 6379
- **Database**: `evolution_db`
- **Database User**: `evolution_user`
- **Database Password**: `evolution_pass`
- **API Key**: `429683C4C977415CAAFCCE10F7D57E11` (default from .env.example)

### Step 4: Start Evolution API

Use the simplified local configuration:

```bash
docker-compose -f docker-compose.local.yaml up -d
```

This will start **4 containers**:
1. **evolution_api** - Main Evolution API service (port 8080)
2. **evolution_frontend** - Web management interface (port 3333)
3. **evolution_postgres** - PostgreSQL database (port 5433)
4. **evolution_redis** - Redis cache (port 6379)

### Step 5: Verify Containers are Running

```bash
docker-compose -f docker-compose.local.yaml ps
```

You should see all containers with status `Up`.

### Step 6: View Logs

```bash
# View all logs
docker-compose -f docker-compose.local.yaml logs -f

# View API logs only
docker-compose -f docker-compose.local.yaml logs -f api

# View database logs
docker-compose -f docker-compose.local.yaml logs -f evolution-postgres
```

### Step 7: Verify API is Working

```bash
# Test API connectivity
curl -H "apikey: 429683C4C977415CAAFCCE10F7D57E11" \
  http://localhost:8080/instance/fetchInstances

# Should return: []
```

**Note**: You may see "redis disconnected" warnings in the logs. This is **normal and can be ignored**. The API uses filesystem caching as a fallback and works perfectly. See [REDIS_FIX.md](./REDIS_FIX.md) for details.

## Accessing Evolution API

### API Endpoints

- **API Base URL**: http://localhost:8080
- **API Documentation**: http://localhost:8080/docs (if available)
- **Health Check**: http://localhost:8080/health (if available)

### Web Management Interface

- **Frontend URL**: http://localhost:3333
- Access the web interface to manage WhatsApp instances

### API Authentication

All API requests require the `apikey` header:

```bash
curl -H "apikey: 429683C4C977415CAAFCCE10F7D57E11" http://localhost:8080/instance/fetchInstances
```

## Common Operations

### Stop Containers

```bash
docker-compose -f docker-compose.local.yaml down
```

### Stop and Remove Data (Clean Restart)

```bash
docker-compose -f docker-compose.local.yaml down -v
```

**Warning**: This removes all data including WhatsApp instances and database!

### Restart Containers

```bash
docker-compose -f docker-compose.local.yaml restart
```

### Restart Only API

```bash
docker-compose -f docker-compose.local.yaml restart api
```

### Update to Latest Version

```bash
docker-compose -f docker-compose.local.yaml pull
docker-compose -f docker-compose.local.yaml up -d
```

## Database Access

### Connect to PostgreSQL

```bash
# Using Docker exec
docker exec -it evolution_postgres psql -U evolution_user -d evolution_db

# Using psql from host (if installed)
psql -h localhost -p 5433 -U evolution_user -d evolution_db
# Password: evolution_pass
```

### Database Management

The database will be automatically migrated when the API starts. Data is persisted in Docker volumes.

## Redis Access

### Connect to Redis

```bash
# Using Docker exec
docker exec -it evolution_redis redis-cli

# Test connection
docker exec -it evolution_redis redis-cli ping
# Should return: PONG
```

## Troubleshooting

### Problem: Containers Won't Start

**Check Docker Desktop is running**:
```bash
docker info
```

**Check port conflicts**:
```bash
lsof -i :8080
lsof -i :3333
lsof -i :5433
lsof -i :6379
```

### Problem: Database Connection Failed

**Check database is running**:
```bash
docker-compose -f docker-compose.local.yaml logs evolution-postgres
```

**Verify connection string in logs**:
```bash
docker-compose -f docker-compose.local.yaml logs api | grep DATABASE
```

### Problem: API Returns 500 Error

**Check API logs**:
```bash
docker-compose -f docker-compose.local.yaml logs -f api
```

**Restart API container**:
```bash
docker-compose -f docker-compose.local.yaml restart api
```

### Problem: Frontend Not Loading

**Check frontend logs**:
```bash
docker-compose -f docker-compose.local.yaml logs -f frontend
```

**Verify frontend can reach API**:
```bash
docker exec -it evolution_frontend curl http://api:8080/health
```

## Configuration Changes

To modify settings:

1. Edit `.env` file
2. Restart containers:
   ```bash
   docker-compose -f docker-compose.local.yaml down
   docker-compose -f docker-compose.local.yaml up -d
   ```

## Important Environment Variables

Located in `.env` file:

### Server Configuration
- `SERVER_PORT=8080` - API server port
- `SERVER_URL=http://localhost:8080` - Public API URL

### Database Configuration
- `DATABASE_PROVIDER=postgresql` - Database type
- `DATABASE_CONNECTION_URI` - PostgreSQL connection string
- `POSTGRES_DATABASE=evolution_db` - Database name
- `POSTGRES_USERNAME=evolution_user` - Database user
- `POSTGRES_PASSWORD=evolution_pass` - Database password

### Cache Configuration
- `CACHE_REDIS_ENABLED=true` - Enable Redis caching
- `CACHE_REDIS_URI=redis://evolution-redis:6379/6` - Redis connection

### Authentication
- `AUTHENTICATION_API_KEY=429683C4C977415CAAFCCE10F7D57E11` - Global API key

### Logging
- `LOG_LEVEL=ERROR,WARN,DEBUG,INFO,LOG,VERBOSE,DARK,WEBHOOKS,WEBSOCKET`

## Integration with Resérvalo Backend (Future M2)

When integrating Evolution API with Resérvalo Backend:

1. **Update Resérvalo Backend `.env`**:
   ```bash
   EVOLUTION_API_URL=http://localhost:8080
   EVOLUTION_API_KEY=429683C4C977415CAAFCCE10F7D57E11
   ```

2. **Create WhatsApp instances via API**:
   ```bash
   curl -X POST http://localhost:8080/instance/create \
     -H "apikey: 429683C4C977415CAAFCCE10F7D57E11" \
     -H "Content-Type: application/json" \
     -d '{
       "instanceName": "tenant-123",
       "qrcode": true
     }'
   ```

3. **Configure webhooks** to send events to Resérvalo Backend:
   ```bash
   curl -X POST http://localhost:8080/webhook/set/tenant-123 \
     -H "apikey: 429683C4C977415CAAFCCE10F7D57E11" \
     -H "Content-Type: application/json" \
     -d '{
       "url": "http://host.docker.internal:3000/webhooks/whatsapp",
       "events": ["messages.upsert", "connection.update"]
     }'
   ```

## Docker Compose Files

Two docker-compose files are available:

### 1. `docker-compose.local.yaml` (Recommended for Development)

- **Simplified configuration** for local development
- **All ports exposed** for easy access and debugging
- **No external networks** required
- **Database port 5433** to avoid conflicts with Resérvalo Backend (5432)
- **Frontend port 3333** to avoid conflicts with Resérvalo Backend (3000)

### 2. `docker-compose.yaml` (Original/Production)

- Includes `dokploy-network` for production deployments
- API only accessible via localhost (127.0.0.1:8080)
- Frontend on port 3000

## Data Persistence

All data is stored in Docker volumes:

- `evolution_instances` - WhatsApp instance data and session files
- `evolution_redis` - Redis cache data
- `postgres_data` - PostgreSQL database files

### Backup Data

```bash
# Backup database
docker exec evolution_postgres pg_dump -U evolution_user evolution_db > evolution_backup.sql

# Backup volumes
docker run --rm -v evolution_instances:/data -v $(pwd):/backup alpine tar czf /backup/instances_backup.tar.gz -C /data .
```

### Restore Data

```bash
# Restore database
cat evolution_backup.sql | docker exec -i evolution_postgres psql -U evolution_user -d evolution_db

# Restore volumes
docker run --rm -v evolution_instances:/data -v $(pwd):/backup alpine tar xzf /backup/instances_backup.tar.gz -C /data
```

## Next Steps

1. ✅ Start Evolution API containers
2. ✅ Access web interface at http://localhost:3333
3. ✅ Create a WhatsApp instance
4. ✅ Scan QR code to connect WhatsApp
5. 📋 Test sending/receiving messages
6. 📋 Configure webhooks for Resérvalo integration (M2)

## Resources

- **Official Documentation**: https://doc.evolution-api.com
- **Docker Hub**: https://hub.docker.com/r/evoapicloud/evolution-api
- **GitHub**: https://github.com/EvolutionAPI/evolution-api
- **Postman Collection**: https://evolution-api.com/postman

## Support

- **WhatsApp Group**: https://evolution-api.com/whatsapp
- **Discord Community**: https://evolution-api.com/discord
- **GitHub Issues**: https://github.com/EvolutionAPI/evolution-api/issues
