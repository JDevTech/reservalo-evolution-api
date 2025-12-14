# Database & Cache Visualization Guide

This guide shows you how to visualize and interact with PostgreSQL and Redis data using GUI tools.

## Overview

Your Evolution API setup now includes:

| Service | URL | Purpose |
|---------|-----|---------|
| **pgAdmin** | http://localhost:5050 | PostgreSQL database management |
| **RedisInsight** | http://localhost:5540 | Redis cache visualization |
| **PostgreSQL** | localhost:5433 | Database server |
| **Redis** | localhost:6379 | Cache server |

---

## 1️⃣ Visualizing PostgreSQL Data

### Option A: pgAdmin (Web UI) - Recommended

**Access**: http://localhost:5050

**Login Credentials**:
- Email: `admin@example.com`
- Password: `admin123`

**First-Time Setup**:

1. Open http://localhost:5050 in your browser
2. Login with the credentials above
3. Right-click "Servers" → "Register" → "Server"
4. Configure the connection:

**General Tab**:
- Name: `Evolution API`

**Connection Tab**:
- Host name/address: `evolution_postgres`
- Port: `5432` (internal Docker port)
- Maintenance database: `evolution_db`
- Username: `evolution_user`
- Password: `evolution_pass`
- Save password: ✅ Yes

5. Click "Save"

**Navigate Data**:
- Servers → Evolution API → Databases → evolution_db → Schemas → evolution_api → Tables
- Right-click any table → View/Edit Data → All Rows

**Common Tables to Explore**:
- `Instance` - WhatsApp instances
- `Message` - Messages sent/received
- `Contact` - WhatsApp contacts
- `Chat` - Conversations
- `Webhook` - Webhook configurations

---

### Option B: Prisma Studio (Alternative)

If you prefer Prisma's visual interface:

```bash
cd /Users/jd/Desktop/Projects/Cholao/Development/Reservalo/Backend/reservalo-evolution-api

# Open Prisma Studio
npx prisma studio
```

This opens at http://localhost:5555 with a clean UI for browsing tables.

---

### Option C: Command Line (Quick Queries)

For quick SQL queries:

```bash
# Connect to PostgreSQL
docker exec -it evolution_postgres psql -U evolution_user -d evolution_db

# Example queries
\dt evolution_api.*          -- List all tables
SELECT * FROM evolution_api."Instance";
SELECT * FROM evolution_api."Message" LIMIT 10;

# Exit
\q
```

---

## 2️⃣ Visualizing Redis Data

### RedisInsight (Web UI) - Recommended

**Access**: http://localhost:5540

**First-Time Setup**:

1. Open http://localhost:5540 in your browser
2. Click "Add Redis Database"
3. Configure the connection:
   - Host: `redis`
   - Port: `6379`
   - Database Alias: `Evolution API Cache`
   - Username: (leave empty)
   - Password: (leave empty)
4. Click "Add Database"

**Navigate Data**:
- Click on "Evolution API Cache"
- Browse keys with prefix: `evolution:*`
- Click any key to view its value

**Common Redis Keys**:
- `evolution:instance:*` - Instance data
- `evolution:groups:*` - WhatsApp group cache
- `evolution:baileys:*` - Baileys library cache

**Features**:
- 🔍 Search keys by pattern
- 📊 View key statistics
- 🗑️ Delete keys
- ⏱️ Check TTL (time to live)
- 📝 Edit values

---

### Command Line (Quick Access)

For quick Redis commands:

```bash
# Connect to Redis CLI
docker exec -it redis redis-cli

# Browse keys
KEYS evolution:*           # List all Evolution keys
GET evolution:instance:ID  # Get specific instance data
TTL evolution:instance:ID  # Check expiration time
SCAN 0 MATCH evolution:*   # Scan keys (better for large datasets)

# Statistics
INFO stats                 # Cache statistics
DBSIZE                     # Number of keys

# Exit
exit
```

---

## 3️⃣ Monitoring Cache Performance

### Check Redis Cache Hit Rate

```bash
docker exec redis redis-cli INFO stats | grep keyspace
```

### Monitor Real-Time Redis Activity

```bash
# Watch all Redis commands in real-time
docker exec -it redis redis-cli MONITOR
```

Press `Ctrl+C` to stop monitoring.

---

## 4️⃣ Common Use Cases

### Use Case 1: Debug WhatsApp Instance

**Scenario**: Check if an instance is stored in cache

```bash
# Via Redis CLI
docker exec redis redis-cli KEYS "evolution:instance:*"

# Via RedisInsight
# Browse to http://localhost:5540 → Search "evolution:instance:*"
```

### Use Case 2: View Message History

**Scenario**: See all messages for debugging

**Via pgAdmin**:
1. Open http://localhost:5050
2. Navigate: Evolution API → evolution_db → evolution_api → Tables → Message
3. Right-click → View/Edit Data → All Rows
4. Filter by instance, contact, or date

**Via SQL**:
```bash
docker exec -it evolution_postgres psql -U evolution_user -d evolution_db -c \
  "SELECT * FROM evolution_api.\"Message\" ORDER BY \"messageTimestamp\" DESC LIMIT 10;"
```

### Use Case 3: Clear Cache for Instance

**Scenario**: Force refresh instance data

```bash
# Find instance keys
docker exec redis redis-cli KEYS "evolution:instance:YOUR_INSTANCE_ID"

# Delete specific key
docker exec redis redis-cli DEL "evolution:instance:YOUR_INSTANCE_ID"

# Or via RedisInsight - Browse and click delete button
```

### Use Case 4: Export Database for Backup

**Via pgAdmin**:
1. Right-click on `evolution_db` → Backup
2. Choose format: Custom or Tar
3. Download the backup file

**Via Command Line**:
```bash
docker exec evolution_postgres pg_dump -U evolution_user evolution_db > evolution_backup_$(date +%Y%m%d).sql
```

---

## 5️⃣ Quick Reference

### Service URLs

```bash
# Web Interfaces
http://localhost:5050   # pgAdmin (PostgreSQL)
http://localhost:5540   # RedisInsight (Redis)
http://localhost:8080   # Evolution API

# Direct Connections
localhost:5433          # PostgreSQL (from host)
localhost:6379          # Redis (from host)
```

### Credentials

**pgAdmin Login**:
- Email: `admin@example.com`
- Password: `admin123`

**PostgreSQL Connection**:
- Host: `evolution_postgres` (Docker) or `localhost` (host machine)
- Port: `5432` (Docker) or `5433` (host machine)
- Database: `evolution_db`
- Username: `evolution_user`
- Password: `evolution_pass`

**Redis Connection**:
- Host: `redis` (Docker) or `localhost` (host machine)
- Port: `6379`
- No password required

---

## 6️⃣ Troubleshooting

### pgAdmin won't load

```bash
# Check logs
docker logs evolution_pgadmin

# Restart container
docker-compose -f docker-compose.local.yaml restart pgadmin
```

### RedisInsight shows empty

**Check Redis has data**:
```bash
docker exec redis redis-cli DBSIZE
```

If zero, no data is cached yet. Create a WhatsApp instance to populate cache.

### Can't connect from pgAdmin

**Make sure you use the Docker network hostname**:
- ✅ Host: `evolution_postgres`
- ❌ Host: `localhost` (this won't work inside Docker)

---

## 7️⃣ Production Considerations

**Security**:
- Change default pgAdmin password in production
- Add Redis password authentication
- Restrict pgAdmin/RedisInsight to internal network only

**Performance**:
- Monitor Redis memory usage: `docker exec redis redis-cli INFO memory`
- Set Redis max memory limit in production
- Regular PostgreSQL vacuum: `VACUUM ANALYZE;`

**Backups**:
- Schedule automated database backups
- Redis persistence is enabled (AOF mode)
- Backup volumes: `postgres_data`, `evolution_redis`, `pgadmin_data`, `redisinsight_data`

---

## Summary

You now have full visibility into your Evolution API data:

✅ **pgAdmin** - Browse PostgreSQL tables visually
✅ **RedisInsight** - Explore Redis cache in real-time
✅ **CLI Access** - Quick commands for both databases
✅ **Monitoring** - Track performance and cache hits

**Next Steps**:
1. Open http://localhost:5050 and set up pgAdmin
2. Open http://localhost:5540 and connect to Redis
3. Create a WhatsApp instance and watch data populate
4. Explore the tables and cache keys

Happy debugging! 🚀
