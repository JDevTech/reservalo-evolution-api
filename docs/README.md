# Evolution API Documentation

Welcome to the Evolution API documentation! This guide helps you navigate all available documentation.

## 📖 Documentation Structure

```
docs/
├── README.md                           # This file
├── AGENTS.md                          # AI development agent guidelines
├── CLAUDE.md                          # Claude AI quick reference
├── setup/                             # Setup & Configuration
│   ├── QUICK_START.md                # Get started in 3 steps
│   ├── DOCKER_SETUP.md               # Complete Docker setup
│   └── REDIS_FIX.md                  # Redis troubleshooting (RESOLVED)
├── api/                              # API Usage
│   └── FRONTEND_ALTERNATIVES.md      # Using the API (Postman/curl)
└── database/                         # Database & Visualization
    └── DATABASE_VISUALIZATION.md     # pgAdmin & RedisInsight setup
```

## 🚀 Quick Links

### Getting Started
New to Evolution API? Start here:

1. [Quick Start Guide](setup/QUICK_START.md) - Get running in 3 steps
2. [Docker Setup](setup/DOCKER_SETUP.md) - Detailed setup instructions
3. [Frontend Alternatives](api/FRONTEND_ALTERNATIVES.md) - How to use the API

### Common Tasks

**Create a WhatsApp Instance:**
```bash
curl -X POST http://localhost:8080/instance/create \
  -H "apikey: 429683C4C977415CAAFCCE10F7D57E11" \
  -H "Content-Type: application/json" \
  -d '{
    "instanceName": "my-business",
    "qrcode": true,
    "integration": "WHATSAPP-BAILEYS"
  }'
```

See [Frontend Alternatives](api/FRONTEND_ALTERNATIVES.md) for more examples.

**View Database Data:**
- PostgreSQL: http://localhost:5050 (pgAdmin)
- Redis: http://localhost:5540 (RedisInsight)

See [Database Visualization](database/DATABASE_VISUALIZATION.md) for setup.

### For Developers

**Working with AI:**
- [AGENTS.md](AGENTS.md) - Comprehensive guidelines for AI development agents
- [CLAUDE.md](CLAUDE.md) - Quick reference for Claude AI

**Troubleshooting:**
- [Redis Fix](setup/REDIS_FIX.md) - Redis connection issue (✅ RESOLVED)

## 📂 Project Root Files

These important files remain in the project root:

- `README.md` - Project overview and main documentation
- `CHANGELOG.md` - Version history and changes
- `SECURITY.md` - Security policies and reporting

## 🔗 External Resources

- [Official Documentation](https://doc.evolution-api.com)
- [Postman Collection](https://evolution-api.com/postman)
- [WhatsApp Community](https://evolution-api.com/whatsapp)
- [Discord Community](https://evolution-api.com/discord)
- [Feature Requests](https://evolutionapi.canny.io/feature-requests)

## 📊 Documentation Coverage

| Category | Files | Status |
|----------|-------|--------|
| Setup & Configuration | 3 | ✅ Complete |
| API Usage | 1 | ✅ Complete |
| Database & Visualization | 1 | ✅ Complete |
| AI Development | 2 | ✅ Complete |

## 🆘 Need Help?

1. Check the [Quick Start Guide](setup/QUICK_START.md) first
2. Review [Docker Setup](setup/DOCKER_SETUP.md) for detailed troubleshooting
3. Join the [Discord Community](https://evolution-api.com/discord)
4. Report issues on [GitHub](https://github.com/EvolutionAPI/evolution-api/issues)

---

Last updated: December 2025
