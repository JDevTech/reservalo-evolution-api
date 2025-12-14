# Evolution API - Frontend Alternatives

## Issue Summary

The Evolution Manager frontend (`evoapicloud/evolution-manager:latest`) has a **broken nginx configuration** that prevents it from starting:

```
nginx: [emerg] invalid value "must-revalidate" in /etc/nginx/conf.d/nginx.conf:11
```

This is a **bug in the Docker image** that we cannot fix without rebuilding the image.

## Impact

- ⚠️ **Web UI is unavailable**
- ✅ **Evolution API works perfectly** (100% functional)
- ✅ **All WhatsApp features available via REST API**

## Recommended Solutions

### **Solution 1: Use Postman (Best for Testing)** ⭐

**What**: Postman is a popular API testing tool with a visual interface.

**Setup**:
1. Download Postman: https://www.postman.com/downloads/
2. Import Evolution API collection: https://evolution-api.com/postman
3. Set up environment variables:
   - `baseUrl`: `http://localhost:8080`
   - `apikey`: `429683C4C977415CAAFCCE10F7D57E11`

**Pros**:
- ✅ Visual interface (similar to web UI)
- ✅ Complete API documentation
- ✅ Save requests for reuse
- ✅ Test automation capabilities
- ✅ Official Evolution API collection

**Example Workflow**:
1. Open Postman
2. Create new request
3. POST `http://localhost:8080/instance/create`
4. Add header: `apikey: 429683C4C977415CAAFCCE10F7D57E11`
5. Add JSON body:
   ```json
   {
     "instanceName": "my-whatsapp",
     "qrcode": true
   }
   ```
6. Send request → Get QR code → Scan with WhatsApp

---

### **Solution 2: Use curl (Best for Scripting)**

**What**: Command-line tool for making HTTP requests.

**Create WhatsApp Instance**:
```bash
curl -X POST http://localhost:8080/instance/create \
  -H "apikey: 429683C4C977415CAAFCCE10F7D57E11" \
  -H "Content-Type: application/json" \
  -d '{
    "instanceName": "my-instance",
    "qrcode": true,
    "integration": "WHATSAPP-BAILEYS"
  }'
```

**⚠️ Important**: The `"integration": "WHATSAPP-BAILEYS"` field is **required** or you'll get a "Bad Request: Invalid integration" error.

**Connect Instance (Get QR Code)**:
```bash
curl -X GET http://localhost:8080/instance/connect/my-instance \
  -H "apikey: 429683C4C977415CAAFCCE10F7D57E11"
```

**Check Instance Status**:
```bash
curl -X GET http://localhost:8080/instance/connectionState/my-instance \
  -H "apikey: 429683C4C977415CAAFCCE10F7D57E11"
```

**Send Message**:
```bash
curl -X POST http://localhost:8080/message/sendText/my-instance \
  -H "apikey: 429683C4C977415CAAFCCE10F7D57E11" \
  -H "Content-Type: application/json" \
  -d '{
    "number": "573001234567",
    "text": "Hello from Evolution API!"
  }'
```

**List All Instances**:
```bash
curl -X GET http://localhost:8080/instance/fetchInstances \
  -H "apikey: 429683C4C977415CAAFCCE10F7D57E11"
```

**Pros**:
- ✅ No additional software needed
- ✅ Easy to script
- ✅ Works on all platforms
- ✅ Good for automation

**Cons**:
- ❌ Less user-friendly than GUI
- ❌ Need to format JSON manually

---

### **Solution 3: Build Custom Simple Frontend**

**What**: Create a minimal web interface for Evolution API.

**Quick HTML Example**:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Evolution API Manager</title>
</head>
<body>
    <h1>Evolution API Simple Manager</h1>

    <h2>Create Instance</h2>
    <input id="instanceName" placeholder="Instance Name" />
    <button onclick="createInstance()">Create</button>

    <h2>List Instances</h2>
    <button onclick="listInstances()">Fetch</button>
    <pre id="output"></pre>

    <script>
        const API_URL = 'http://localhost:8080';
        const API_KEY = '429683C4C977415CAAFCCE10F7D57E11';

        async function createInstance() {
            const name = document.getElementById('instanceName').value;
            const response = await fetch(`${API_URL}/instance/create`, {
                method: 'POST',
                headers: {
                    'apikey': API_KEY,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    instanceName: name,
                    qrcode: true
                })
            });
            const data = await response.json();
            document.getElementById('output').textContent = JSON.stringify(data, null, 2);
        }

        async function listInstances() {
            const response = await fetch(`${API_URL}/instance/fetchInstances`, {
                headers: { 'apikey': API_KEY }
            });
            const data = await response.json();
            document.getElementById('output').textContent = JSON.stringify(data, null, 2);
        }
    </script>
</body>
</html>
```

Save as `evolution-manager.html` and open in browser.

**Pros**:
- ✅ Custom to your needs
- ✅ No dependencies
- ✅ Full control

**Cons**:
- ❌ Requires development time
- ❌ Need to maintain

---

### **Solution 4: Use Third-Party Evolution API Clients**

**What**: Community-built tools for Evolution API.

**Options**:
- Check Evolution API Discord/GitHub for community tools
- Look for browser extensions
- Search for "Evolution API client" tools

**Note**: Verify security before using third-party tools.

---

### **Solution 5: Wait for Official Fix**

**What**: Monitor Evolution API repository for frontend image fix.

**How**:
1. Check GitHub issues: https://github.com/EvolutionAPI/evolution-api/issues
2. Search for "nginx" or "evolution-manager" issues
3. Watch for new image releases
4. Update when fixed version available

**Check if issue reported**:
```bash
# Search GitHub issues
# Look for: "evolution-manager nginx config error"
```

---

## Integration with Resérvalo Backend (M2)

For your **Resérvalo Backend** integration, you'll use the REST API directly:

```typescript
// src/infrastructure/whatsapp/evolution-api.service.ts
import axios from 'axios';

export class EvolutionApiService {
  private readonly baseURL = process.env.EVOLUTION_API_URL; // http://localhost:8080
  private readonly apiKey = process.env.EVOLUTION_API_KEY;

  async createInstance(tenantId: string) {
    const response = await axios.post(
      `${this.baseURL}/instance/create`,
      {
        instanceName: `tenant-${tenantId}`,
        qrcode: true,
      },
      {
        headers: { apikey: this.apiKey },
      }
    );
    return response.data;
  }

  async sendMessage(instanceName: string, number: string, text: string) {
    const response = await axios.post(
      `${this.baseURL}/message/sendText/${instanceName}`,
      { number, text },
      {
        headers: { apikey: this.apiKey },
      }
    );
    return response.data;
  }
}
```

**You don't need the frontend** - your backend will interact with Evolution API programmatically!

---

## Current Setup Status

**Disabled Frontend** ✅
- Frontend service commented out in `docker-compose.local.yaml`
- No more restart loops or error spam
- Clean logs

**Working Services** ✅
- Evolution API: http://localhost:8080
- PostgreSQL: port 5433
- Redis: port 6379 (for future use)

---

## Quick Reference: Common API Calls

### Create Instance
```bash
POST http://localhost:8080/instance/create
Headers: apikey: 429683C4C977415CAAFCCE10F7D57E11
Body: {"instanceName": "test", "qrcode": true}
```

### Get QR Code
```bash
GET http://localhost:8080/instance/connect/test
Headers: apikey: 429683C4C977415CAAFCCE10F7D57E11
```

### Send Message
```bash
POST http://localhost:8080/message/sendText/test
Headers: apikey: 429683C4C977415CAAFCCE10F7D57E11
Body: {"number": "573001234567", "text": "Hello!"}
```

### List Instances
```bash
GET http://localhost:8080/instance/fetchInstances
Headers: apikey: 429683C4C977415CAAFCCE10F7D57E11
```

### Delete Instance
```bash
DELETE http://localhost:8080/instance/delete/test
Headers: apikey: 429683C4C977415CAAFCCE10F7D57E11
```

---

## Documentation Resources

- **Official API Docs**: https://doc.evolution-api.com
- **Postman Collection**: https://evolution-api.com/postman
- **GitHub Repository**: https://github.com/EvolutionAPI/evolution-api
- **Discord Community**: https://evolution-api.com/discord

---

## Recommendation for Resérvalo Project

**Use Postman for development/testing** ⭐
- Download Postman
- Import Evolution API collection
- Test WhatsApp features manually

**Use REST API for integration** ⭐
- Build Evolution API service in Resérvalo Backend
- No dependency on frontend
- Better for automation and production

**Frontend is optional** - You're building your own Resérvalo Frontend that will manage WhatsApp through your backend!

---

**Bottom Line**: The broken frontend is **not a blocker**. Evolution API is fully functional via REST API, which is exactly what you'll use for Resérvalo integration (M2).
