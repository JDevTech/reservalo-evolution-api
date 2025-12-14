# Evolution API - Quick Start 🚀

Get Evolution API running in 3 steps!

## 1️⃣ Start Docker Desktop

Make sure Docker Desktop is running on your Mac.

## 2️⃣ Start Evolution API

```bash
cd /Users/jd/Desktop/Projects/Cholao/Development/Reservalo/Backend/reservalo-evolution-api

docker-compose -f docker-compose.local.yaml up -d
```

## 3️⃣ Access the Services

### Evolution API
- **API**: http://localhost:8080
- **API Key**: `429683C4C977415CAAFCCE10F7D57E11`
- **Frontend**: Disabled (see [Frontend Alternatives](../api/FRONTEND_ALTERNATIVES.md))

### Database Visualization
- **pgAdmin**: http://localhost:5050 (PostgreSQL GUI)
  - Email: `admin@example.com`
  - Password: `admin123`
- **RedisInsight**: http://localhost:5540 (Redis GUI)

See [Database Visualization Guide](../database/DATABASE_VISUALIZATION.md) for setup instructions.

**Note**: Use Postman or curl to interact with the API. Download Postman collection: https://evolution-api.com/postman

---

## Common Commands

### View Logs
```bash
docker-compose -f docker-compose.local.yaml logs -f
```

### Stop Everything
```bash
docker-compose -f docker-compose.local.yaml down
```

### Restart API
```bash
docker-compose -f docker-compose.local.yaml restart api
```

### Check Status
```bash
docker-compose -f docker-compose.local.yaml ps
```

---

## Test API Connection

```bash
curl -H "apikey: 429683C4C977415CAAFCCE10F7D57E11" \
  http://localhost:8080/instance/fetchInstances
```

---

## Ports

- **8080** - Evolution API
- **5050** - pgAdmin (PostgreSQL GUI)
- **5540** - RedisInsight (Redis GUI)
- **5433** - PostgreSQL (mapped to avoid conflict with Resérvalo Backend)
- **6379** - Redis

---

## Need Help?

See [DOCKER_SETUP.md](./DOCKER_SETUP.md) for detailed instructions and troubleshooting.
