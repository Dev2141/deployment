# 🚀 Complete Connection & Deployment Guide

This guide walks you through **everything** from start to finish, with exact steps and expected outputs.

---

## 📊 The Big Picture

```
┌──────────────────────────────────────────────────────────────────┐
│                    YOUR DEPLOYED SYSTEM                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. You push code → GitHub                                      │
│       ↓                                                          │
│  2. Vercel auto-deploys → https://xxx.vercel.app (Frontend)    │
│       ↓                                                          │
│  3. Render auto-deploys → https://xxx.onrender.com (Backend)   │
│       ↓                                                          │
│  4. Backend connects to:                                        │
│       ├─ Neo4j Aura (stores entities & relationships)          │
│       ├─ Pinecone (stores embeddings for search)               │
│       └─ LLM API (generates answers)                           │
│       ↓                                                          │
│  5. Frontend calls Backend API                                 │
│       ↓                                                          │
│  6. User gets answers with graph + vector search               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Full Deployment Sequence

### PHASE 1: SETUP EXTERNAL SERVICES (30 minutes)

#### Step 1: Neo4j Aura (Graph Database)

**What it is**: Stores entities (Person, Organization, Project) and relationships between them

**Setup:**
1. Go to https://neo4j.com/cloud/aura
2. Click "Create"
3. Choose "Free Tier"
4. Select region closest to you
5. Click "Create Instance"
6. **IMPORTANT**: Copy these immediately (save in text file):
   ```
   Connection URI: neo4j+s://xxxxxxxx.databases.neo4j.io
   Username: neo4j
   Password: [AUTO-GENERATED - CHECK YOUR EMAIL]
   ```

**Verify it works:**
- Go to "Open" → Neo4j Browser
- Run: `RETURN "Neo4j is working!" as message`
- Should see: "Neo4j is working!"

✅ **Neo4j ready**

---

#### Step 2: Pinecone (Vector Database)

**What it is**: Stores embeddings (mathematical representations) of email text for similarity search

**Setup:**
1. Go to https://www.pinecone.io
2. Sign up with GitHub
3. Go to Dashboard
4. Click "Create Index"
5. Fill in:
   ```
   Name: email-knowledge-graph
   Dimension: 1024
   Metric: cosine
   ```
6. Click "Create"
7. Go to "API Keys"
8. **Copy API Key** (you'll need this)

**Verify it works:**
```bash
curl -X GET "https://api.pinecone.io/indexes" \
  -H "Api-Key: YOUR_PINECONE_API_KEY"
```

✅ **Pinecone ready**

---

#### Step 3: LLM API Key (Choose ONE)

**Option A: Groq (RECOMMENDED - Fastest, Free)**

1. Go to https://console.groq.com
2. Sign up
3. Click "API Keys"
4. Click "Create API Key"
5. Copy the key

**Option B: OpenAI (Most popular, Paid)**

1. Go to https://platform.openai.com
2. Sign up
3. Add payment method (you get free credits)
4. Go to "API Keys"
5. Click "Create new"
6. Copy the key

**Option C: Ollama (Local, Free)**

1. Install from https://ollama.ai
2. Start: `ollama serve`
3. Download model: `ollama pull llama2`
4. Note: This runs locally, not in cloud

✅ **LLM ready**

---

### PHASE 2: DEPLOY BACKEND (Render) - 10 minutes

#### Step 1: Prepare Backend

In deployment repo on your computer:

```bash
cd /c/Users/padha/Desktop/deployment

# Make sure api.py is there
ls api.py
# Output: api.py ✓

# Check requirements.txt has Flask, CORS, Gunicorn
cat requirements.txt | head -10
# Output should show:
# flask>=2.3.0
# flask-cors>=4.0.0
# gunicorn>=21.0.0
```

#### Step 2: Go to Render

1. Go to https://render.com
2. Dashboard → Click **New** → **Web Service**
3. Select `deployment` repository

#### Step 3: Configure Service

Fill in:
```
Name:              vidzai-api
Environment:       Python 3
Build Command:     pip install -r requirements.txt
Start Command:     gunicorn api:app
Region:            Choose US/EU (closest to you)
```

#### Step 4: Add Environment Variables

In Render → **Environment** tab, add these (copy from Step 1):

```
NEO4J_URI=neo4j+s://[YOUR_URI_FROM_NEO4J]
NEO4J_USER=neo4j
NEO4J_PASSWORD=[YOUR_PASSWORD_FROM_NEO4J]

PINECONE_API_KEY=[YOUR_PINECONE_API_KEY]
PINECONE_INDEX=email-knowledge-graph
PINECONE_ENV=prod

LLAMA_API_KEY=[YOUR_LLM_API_KEY]
MODEL_NAME=gpt-oss:120b-cloud

FLASK_ENV=production
FLASK_DEBUG=false
```

**Example (FAKE - DO NOT USE):**
```
NEO4J_URI=neo4j+s://12345.databases.neo4j.io
NEO4J_USER=neo4j
NEO4J_PASSWORD=aB3xYz9qPw
PINECONE_API_KEY=abcd123-xyz789
PINECONE_INDEX=email-knowledge-graph
PINECONE_ENV=prod
LLAMA_API_KEY=gsk_abc123xyz
MODEL_NAME=gpt-oss:120b-cloud
FLASK_ENV=production
FLASK_DEBUG=false
```

#### Step 5: Deploy

1. Click **Create Web Service**
2. Wait for build (~5-10 minutes)
3. When done, you'll see green checkmark
4. Copy your Render URL: `https://vidzai-api.onrender.com`
5. **SAVE THIS URL**

#### Step 6: Test Backend

In browser console, run:
```javascript
fetch('https://vidzai-api.onrender.com/api/health')
  .then(r => r.json())
  .then(d => console.log(d))
```

Expected output:
```json
{
  "status": "ok",
  "missing_env": [],
  "model": "gpt-oss:120b-cloud",
  "pinecone_index": "email-knowledge-graph"
}
```

✅ **Backend deployed!**

---

### PHASE 3: DEPLOY FRONTEND (Vercel) - 5 minutes

#### Step 1: Go to Vercel

1. Go to https://vercel.com
2. Dashboard → Click **Add New** → **Project**
3. Search for `deployment`
4. Click **Import**

#### Step 2: Configure Build Settings

Vercel should auto-detect:
```
Framework:        Vite
Build Command:    npm install && npm run build
Output Directory: frontend/dist
Root Directory:   frontend
```

If not auto-detected, set manually in **Settings → Build & Deployment**

#### Step 3: Add Environment Variable

1. Go to **Settings** → **Environment Variables**
2. Add:
   ```
   VITE_API_URL = https://vidzai-api.onrender.com
   ```
   (Use the Render URL from Phase 2)

3. Click **Save**

#### Step 4: Deploy

1. Click **Deploy**
2. Wait for build (~2-3 minutes)
3. When done, you'll see URL like: `https://xxxxx.vercel.app`
4. **SAVE THIS URL**

#### Step 5: Test Frontend

1. Open `https://xxxxx.vercel.app`
2. React app should load
3. Open DevTools → Console
4. Should see NO CORS errors

✅ **Frontend deployed!**

---

### PHASE 4: POPULATE DATABASE (Local - 20 minutes)

Now your system is live, but it needs DATA. Data comes from running Milestone-2 and Milestone-3 locally.

#### Step 1: Milestone-2 (Extract Entities & Build Graph)

This reads emails and extracts entities (Person, Organization, Project) and relationships.

**In your terminal:**
```bash
cd /c/Users/padha/Desktop/try_2/AI-Knowledge-Graph-Builder-for-Enterprise-Intelligence

# Copy your Neo4j credentials from Phase 1 into .env
# Edit .env and add:
# NEO4J_URI=neo4j+s://[YOUR_URI]
# NEO4J_USER=neo4j
# NEO4J_PASSWORD=[YOUR_PASSWORD]

python Milestone-2.py
```

**What happens:**
- Reads emails from your dataset
- Extracts entities using LLM
- Writes to Neo4j Aura
- You'll see progress: `Processing email 1 of 1000...`

**Expected output:**
```
✅ Emails processed: 1000
🏷️  Unique entities extracted: 5579
🔗 Semantic relationships built: 13069
```

✅ **Neo4j now has data!**

---

#### Step 2: Milestone-3 (Create Embeddings & Index in Pinecone)

This converts email text to embeddings and stores in Pinecone for vector search.

**In your terminal:**
```bash
cd /c/Users/padha/Desktop/try_2/AI-Knowledge-Graph-Builder-for-Enterprise-Intelligence

# Make sure .env has Pinecone credentials:
# PINECONE_API_KEY=[YOUR_KEY]
# PINECONE_INDEX=email-knowledge-graph

python Milestone-3.py
```

**What happens:**
- Reads emails from Neo4j
- Creates vector embeddings
- Stores in Pinecone
- Takes ~5-10 minutes

**Expected output:**
```
Indexing embeddings: [████████████████████] 100%
✅ 1000 emails indexed in Pinecone
```

✅ **Pinecone now has vectors!**

---

### PHASE 5: TEST END-TO-END CONNECTION

Now everything is connected. Test it:

#### Test 1: Backend Health Check

```bash
curl https://vidzai-api.onrender.com/api/health
```

Expected:
```json
{"status": "ok", "missing_env": [], ...}
```

#### Test 2: Query from Backend

```bash
curl -X POST https://vidzai-api.onrender.com/api/query \
  -H "Content-Type: application/json" \
  -d '{"question": "Who communicated about natural gas?"}'
```

Expected: A JSON response with answer, graph facts, and vector results

#### Test 3: Query from Frontend

1. Open `https://xxxxx.vercel.app`
2. Type: "Who communicated most about energy?"
3. Wait for response
4. Should see:
   - Answer from LLM
   - Graph facts (relationships)
   - Evidence (email snippets)

✅ **Full system working!**

---

## 📋 Connection Map

Here's how everything connects:

```
User Browser
    ↓
Frontend (Vercel)
    ↓ HTTPS
Backend (Render)
    ├─ Questions → LLM API (Groq/OpenAI)
    ├─ Entity Search → Neo4j Aura
    └─ Vector Search → Pinecone

Flow for a single query:
1. User types: "Who communicated about natural gas?"
2. Frontend sends HTTPS POST to: https://vidzai-api.onrender.com/api/query
3. Backend receives question
4. Backend does 3 things in parallel:
   a) Queries Neo4j: "Find Person nodes related to 'natural gas'"
   b) Embeds question & searches Pinecone: "Find similar email snippets"
   c) Looks up LLM API key from environment
5. Backend combines results
6. Backend calls LLM API with context
7. LLM generates answer
8. Backend sends answer back to frontend
9. Frontend displays answer with evidence
```

---

## 🔑 Environment Variables Reference

### What goes WHERE:

#### Render Environment (Backend)
```
NEO4J_URI → Auto-filled from Neo4j Aura
NEO4J_USER → Always "neo4j"
NEO4J_PASSWORD → From Neo4j Aura email
PINECONE_API_KEY → From Pinecone dashboard
PINECONE_INDEX → Always "email-knowledge-graph"
LLAMA_API_KEY → From Groq/OpenAI/Ollama
```

#### Vercel Environment (Frontend)
```
VITE_API_URL → Your Render backend URL
```

#### Local .env (for running Milestones locally)
```
NEO4J_URI → Same as Render
NEO4J_USER → Same as Render
NEO4J_PASSWORD → Same as Render
PINECONE_API_KEY → Same as Render
LLAMA_API_KEY → Same as Render
```

---

## ✅ Final Checklist

- [ ] Neo4j Aura instance created and working
- [ ] Pinecone index created (`email-knowledge-graph`)
- [ ] LLM API key obtained (Groq/OpenAI)
- [ ] Backend deployed to Render with all env vars
- [ ] Backend health check returns `{"status": "ok"}`
- [ ] Frontend deployed to Vercel with `VITE_API_URL`
- [ ] Frontend loads without errors
- [ ] Milestone-2 ran locally and populated Neo4j
- [ ] Milestone-3 ran locally and populated Pinecone
- [ ] Test query from frontend returns answer
- [ ] No CORS errors in browser console

---

## 🚨 Common Issues & Fixes

| Issue | Solution |
|-------|----------|
| Backend 502 error | Cold start - Render free tier sleeps. Retry after 30s |
| CORS errors | Check Render logs, verify CORS config in api.py |
| "Neo4j connection failed" | Verify NEO4J_URI format starts with `neo4j+s://` |
| "Pinecone API error" | Check API key is correct, index named `email-knowledge-graph` |
| No results from query | Run Milestone-2 & 3 locally to populate databases |
| Frontend blank page | Check browser console for errors, check VITE_API_URL |
| "Missing environment variables" | Add all vars to Render dashboard, restart service |

---

## 🎯 URLs to Save

```
GitHub (Deployment):     https://github.com/Dev2141/deployment
GitHub (Main):           https://github.com/Dev2141/AI-Knowledge-Graph-Builder-for-Enterprise-Intelligence

Frontend:                https://your-project.vercel.app
Backend:                 https://vidzai-api.onrender.com
Backend Health:          https://vidzai-api.onrender.com/api/health

Dashboard Links:
Render (Backend):        https://dashboard.render.com
Vercel (Frontend):       https://vercel.com/dashboard
Neo4j Aura:              https://neo4j.com/cloud/aura
Pinecone:                https://app.pinecone.io
```

---

## 📞 How to Troubleshoot

1. **Check Render Logs**:
   - Render Dashboard → Select `vidzai-api` → Logs tab
   - Shows all backend errors in real-time

2. **Check Vercel Logs**:
   - Vercel Dashboard → Select project → Deployments → Click deployment → Logs

3. **Test Backend Directly**:
   ```bash
   curl https://vidzai-api.onrender.com/api/health
   ```

4. **Test Frontend Console**:
   - Open browser DevTools (F12)
   - Console tab
   - Look for errors

5. **Check Database Connections**:
   - Neo4j Browser: https://neo4j.com/cloud/aura → Open Neo4j Browser
   - Pinecone: https://app.pinecone.io → Check stats

---

## 🎓 How Updates Work

**When you make changes:**

1. Edit code locally (in `/c/Users/padha/Desktop/deployment/`)
2. Commit: `git add . && git commit -m "message"`
3. Push: `git push origin main`
4. **Automatic**:
   - Vercel rebuilds frontend (2-3 min)
   - Render rebuilds backend (5-10 min)
   - New version goes live

No manual deployment needed!

---

## 🚀 You're Ready!

Follow these phases in order and everything will connect automatically. The system is designed to just work once all pieces are in place.

**Total time**: ~1-2 hours first time through (mostly waiting for deployments)

**Get started**: Pick Phase 1 and begin! 🎯
