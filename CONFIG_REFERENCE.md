# Configuration References

This file contains configuration snippets and references for deployment setup.

## 1. Vercel Configuration

### vercel.json (Optional - for advanced config)

Place this file at the root of the repository:

```json
{
  "buildCommand": "cd frontend && npm install && npm run build",
  "outputDirectory": "frontend/dist",
  "env": {
    "VITE_API_URL": "@vite_api_url"
  }
}
```

### Environment Variables Format

In Vercel Dashboard → Project Settings → Environment Variables:

```
VITE_API_URL = https://vidzai-api.onrender.com
```

Add for all environments: Production, Preview, Development

## 2. Render Configuration

### render.yaml (Optional - for IaC)

Place this file at the root:

```yaml
services:
  - type: web
    name: vidzai-api
    env: python
    plan: free
    buildCommand: pip install -r requirements.txt
    startCommand: gunicorn api:app
    envVars:
      - key: FLASK_ENV
        value: production
      - key: FLASK_DEBUG
        value: false
      - key: NEO4J_URI
        sync: false
      - key: NEO4J_USER
        value: neo4j
      - key: NEO4J_PASSWORD
        sync: false
      - key: PINECONE_API_KEY
        sync: false
      - key: PINECONE_INDEX
        value: email-knowledge-graph
      - key: LLAMA_API_KEY
        sync: false
      - key: MODEL_NAME
        value: gpt-oss:120b-cloud
```

## 3. Frontend Vite Configuration

The frontend uses Vite. Check `frontend/vite.config.ts`:

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  define: {
    'import.meta.env.VITE_API_URL': JSON.stringify(process.env.VITE_API_URL || 'http://localhost:5000')
  }
})
```

## 4. Flask CORS Configuration

In `api.py` (already configured):

```python
from flask_cors import CORS

app = Flask(__name__)
CORS(app, origins=[
    'https://your-project.vercel.app',
    'http://localhost:3000',  # development
    'http://localhost:5000'   # local backend
])
```

## 5. Environment Variables by Service

### Neo4j Aura Variables

```
NEO4J_URI=neo4j+s://xxxxxxxx.databases.neo4j.io
NEO4J_USER=neo4j
NEO4J_PASSWORD=[PASSWORD_FROM_EMAIL]
NEO4J_DATABASE=neo4j
```

### Pinecone Variables

```
PINECONE_API_KEY=[API_KEY_FROM_DASHBOARD]
PINECONE_INDEX=email-knowledge-graph
PINECONE_ENV=prod
```

### LLM Variables

For Groq:
```
LLAMA_API_KEY=[GROQ_API_KEY]
MODEL_NAME=mixtral-8x7b-32768
```

For OpenAI:
```
OPENAI_API_KEY=[OPENAI_API_KEY]
MODEL_NAME=gpt-4
```

For Ollama (local):
```
OLLAMA_BASE_URL=http://localhost:11434
MODEL_NAME=llama2
```

## 6. Docker Configuration (Optional)

If deploying with Docker, use this `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Copy requirements and install
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY api.py .

# Expose port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/api/health || exit 1

# Run application
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "api:app"]
```

And `docker-compose.yml` for local development:

```yaml
version: '3.8'

services:
  backend:
    build: .
    ports:
      - "5000:5000"
    environment:
      FLASK_ENV: development
      NEO4J_URI: neo4j+s://your-uri
      NEO4J_USER: neo4j
      NEO4J_PASSWORD: password
      PINECONE_API_KEY: key
      PINECONE_INDEX: email-knowledge-graph
      LLAMA_API_KEY: key

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      VITE_API_URL: http://localhost:5000
    depends_on:
      - backend
```

## 7. GitHub Actions Workflow (Optional)

For automatic deployments, create `.github/workflows/deploy.yml`:

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Notify Render
        run: curl ${{ secrets.RENDER_DEPLOY_HOOK }}

      - name: Notify Vercel
        run: curl -X POST ${{ secrets.VERCEL_DEPLOY_HOOK }}
```

## 8. Rate Limits & Quotas

### Vercel Free Tier

- Bandwidth: Unlimited
- Deployments: Unlimited
- Functions: 100GB-hours/month
- Domains: Unlimited with vercel.app

### Render Free Tier

- Hours: 750/month (about 1 instance running 24/7)
- Memory: 0.5GB per service
- CPU: Shared
- Cold starts: 30+ seconds after 15 min inactivity

### Neo4j Aura Free

- Storage: 1 GB
- Instances: 1 free database
- Instances: Limited compute

### Pinecone Free

- Vectors: 100,000 max
- Indexes: 1
- Requests: 100 req/second

## 9. Monitoring & Logs

### Render Logs

```bash
# View backend logs in real-time
# Render Dashboard → Service → Logs
```

### Vercel Analytics

- Dashboard → Analytics
- Shows deployment times, build performance, errors

### Neo4j Monitoring

- Aura Dashboard → Insights tab
- Query performance, database size

### Pinecone Monitoring

- Dashboard → Usage
- Vector count, API usage, errors

---

**All configurations are in place. Ready to deploy! 🚀**
