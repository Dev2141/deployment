# 🚀 Quick Start Deployment Checklist

Complete this checklist to deploy Vidzai to production in order.

---

## Pre-Deployment ✓

- [ ] Repository pushed to GitHub: https://github.com/Dev2141/deployment
- [ ] All files copied from main project
- [ ] DEPLOYMENT.md reviewed
- [ ] .env.example file created
- [ ] requirements.txt updated with Flask, CORS, Gunicorn

---

## Step 1: Create Accounts (15 minutes)

### Vercel
- [ ] Go to https://vercel.com
- [ ] Sign up with GitHub
- [ ] Authorize access to repositories

### Render
- [ ] Go to https://render.com
- [ ] Sign up with GitHub
- [ ] Authorize access to repositories

### Neo4j Aura
- [ ] Go to https://neo4j.com/cloud/aura
- [ ] Create free account
- [ ] Create free instance
- [ ] **Save connection details** (email them to yourself)
  - [ ] URI: `neo4j+s://...`
  - [ ] Password: (auto-generated)

### Pinecone
- [ ] Go to https://www.pinecone.io
- [ ] Sign up with GitHub
- [ ] Create free account
- [ ] **Create index**: `email-knowledge-graph`
  - [ ] Dimension: 1024
  - [ ] Metric: cosine
- [ ] **Copy API Key**

### LLM Service (Choose One)
- [ ] **Option A - Groq (Recommended)**
  - [ ] Go to https://console.groq.com
  - [ ] Sign up
  - [ ] Copy API key

- [ ] **Option B - OpenAI**
  - [ ] Go to https://platform.openai.com
  - [ ] Sign up
  - [ ] Add payment method
  - [ ] Create API key

- [ ] **Option C - Ollama (Local)**
  - [ ] Install from https://ollama.ai
  - [ ] Start Ollama: `ollama serve`

---

## Step 2: Deploy Frontend (Vercel) - 5 minutes

- [ ] Go to https://vercel.com/dashboard
- [ ] Click **Add New** → **Project**
- [ ] Select `deployment` repository
- [ ] Click **Import**

### Build Settings (should auto-detect)
- [ ] Framework Preset: `Vite`
- [ ] Build Command: `npm install && npm run build`
- [ ] Output Directory: `frontend/dist`
- [ ] Root Directory: `frontend`

### Environment Variables
- [ ] Add `VITE_API_URL` = `https://vidzai-api.onrender.com`
  *(You'll update this after deploying backend)*

- [ ] Click **Deploy**
- [ ] Wait for build complete (~2-3 min)
- [ ] **Copy Vercel URL**: `https://xxxxx.vercel.app`

✅ **Frontend deployed!**

---

## Step 3: Deploy Backend (Render) - 10 minutes

- [ ] Go to https://render.com/dashboard
- [ ] Click **New** → **Web Service**
- [ ] Select `deployment` repository

### Configuration
- [ ] **Name**: `vidzai-api`
- [ ] **Environment**: `Python 3`
- [ ] **Build Command**: `pip install -r requirements.txt`
- [ ] **Start Command**: `gunicorn api:app`
- [ ] **Region**: Select closest to you

### Environment Variables
Add these (from Step 1):

```
NEO4J_URI=neo4j+s://[YOUR_URI]
NEO4J_USER=neo4j
NEO4J_PASSWORD=[YOUR_PASSWORD]

PINECONE_API_KEY=[YOUR_PINECONE_KEY]
PINECONE_INDEX=email-knowledge-graph
PINECONE_ENV=prod

LLAMA_API_KEY=[YOUR_LLM_KEY]
MODEL_NAME=gpt-oss:120b-cloud

FLASK_ENV=production
FLASK_DEBUG=false
```

- [ ] Click **Create Web Service**
- [ ] Wait for deployment (~5-10 min)
- [ ] **Copy Render URL**: `https://vidzai-api.onrender.com`

✅ **Backend deployed!**

---

## Step 4: Update Frontend with Backend URL - 2 minutes

- [ ] Go back to Vercel Dashboard
- [ ] Select `deployment` project
- [ ] Go to **Settings** → **Environment Variables**
- [ ] Update `VITE_API_URL`:
  ```
  https://vidzai-api.onrender.com
  ```
- [ ] Click **Save**
- [ ] Auto-redeploys (check Deployments tab)

---

## Step 5: Configure External Services - 10 minutes

### Neo4j Aura
- [ ] Go to https://neo4j.com/cloud/aura
- [ ] Copy your instance connection details
- [ ] Verify connection with `neo4j` browser UI
- [ ] Ready for Milestone-2.py to populate data

### Pinecone
- [ ] Go to https://www.pinecone.io/dashboard
- [ ] Verify index created: `email-knowledge-graph`
- [ ] Copy API key (already in Render env vars)
- [ ] Ready for Milestone-3.py to index embeddings

---

## Step 6: Test Everything - 5 minutes

### Test Backend Health
```bash
curl https://vidzai-api.onrender.com/api/health
```

Expected response:
```json
{"status": "ok", "missing_env": [], ...}
```

### Test Frontend Loads
- [ ] Open `https://xxxxx.vercel.app`
- [ ] React app should load
- [ ] Open DevTools → Console (no CORS errors)

### Test Frontend → Backend Connection
In browser console:
```javascript
fetch('https://vidzai-api.onrender.com/api/health')
  .then(r => r.json())
  .then(d => console.log('✅ Connected:', d))
  .catch(e => console.error('❌ Error:', e))
```

- [ ] Should log success message

### Test Full Query
- [ ] Go to frontend UI
- [ ] Enter test question: "Who communicated about natural gas?"
- [ ] Should get a response
- [ ] Check Network tab to verify API calls

---

## Step 7: Data Population - 15 minutes (Local)

After deployment is working, populate data locally:

### Run Milestone-2 (Entity Extraction)
```bash
# In main project directory
python Milestone-2.py
```
- [ ] Extracts entities and relationships
- [ ] Populates Neo4j database

### Run Milestone-3 (Vector Indexing)
```bash
# In main project directory
python Milestone-3.py
```
- [ ] Creates embeddings
- [ ] Populates Pinecone vectors

---

## Step 8: Verify Production Setup

- [ ] Backend logs show no errors (Render Dashboard → Logs)
- [ ] Frontend builds successfully (Vercel Dashboard → Deployments)
- [ ] Query returns results with graph + vectors
- [ ] No CORS errors in browser console
- [ ] Neo4j shows data (verify in Neo4j browser)
- [ ] Pinecone shows vectors indexed

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Backend 502 error | Cold start on Render - wait 30s then retry |
| CORS errors | Verify CORS config in api.py + restart backend |
| No query results | Run Milestone-2 & 3 locally to populate data |
| Neo4j connection failed | Check URI format & password in env vars |
| Vercel blank page | Check browser console for errors, redeploy |

---

## Final Checklist

- [ ] Frontend URL: https://xxxxx.vercel.app ✅
- [ ] Backend URL: https://vidzai-api.onrender.com ✅
- [ ] Health check returns 200 ✅
- [ ] Query works end-to-end ✅
- [ ] All environment variables set ✅
- [ ] No sensitive data in git ✅
- [ ] README.md updated with prod URLs ✅

---

## Production URLs

Save these:

```
Frontend:  https://xxxxx.vercel.app
Backend:   https://vidzai-api.onrender.com
GitHub:    https://github.com/Dev2141/deployment
```

---

## Next Steps

1. **Monitor deployments**:
   - Vercel Dashboard for frontend
   - Render Dashboard for backend

2. **Auto-redeploy on push**:
   - Both services auto-update when you push to GitHub

3. **Scaling** (if needed):
   - Vercel: Already unlimited
   - Render: Upgrade to paid tier
   - Neo4j: Upgrade storage
   - Pinecone: Upgrade vector quota

---

**🎉 Deployment complete! Vidzai is now live!**
