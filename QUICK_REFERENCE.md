# ⚡ Quick Reference: What Goes Where

## The 8 Steps (TL;DR Version)

### Step 1-4: Create Accounts & Get API Keys (30 min)
```
Neo4j Aura
├─ Website: https://neo4j.com/cloud/aura
├─ Create: Free instance
└─ Save: NEO4J_URI + NEO4J_PASSWORD

Pinecone
├─ Website: https://www.pinecone.io
├─ Create: Index "email-knowledge-graph"
└─ Save: PINECONE_API_KEY

LLM (Pick 1)
├─ Groq: https://console.groq.com → Copy LLAMA_API_KEY
├─ OpenAI: https://platform.openai.com → Copy OPENAI_API_KEY
└─ Ollama: Install locally
```

### Step 5: Deploy Backend (10 min)
```
Website: https://render.com
Action:  New Web Service → deployment repo

Configure:
├─ Name: vidzai-api
├─ Build: pip install -r requirements.txt
├─ Start: gunicorn api:app
└─ Add all environment variables (from Step 1-4)

Result: https://vidzai-api.onrender.com
```

### Step 6: Deploy Frontend (5 min)
```
Website: https://vercel.com
Action:  New Project → deployment repo

Configure:
├─ Build: npm install && npm run build
├─ Output: frontend/dist
└─ Env: VITE_API_URL = https://vidzai-api.onrender.com

Result: https://xxxxx.vercel.app
```

### Step 7: Populate Databases (20 min - LOCAL ONLY)
```
Terminal:
cd /c/Users/padha/Desktop/try_2/AI-Knowledge-Graph-Builder-for-Enterprise-Intelligence

Update .env with credentials from Step 1-4

Run:
python Milestone-2.py    # Extracts entities → Neo4j
python Milestone-3.py    # Creates embeddings → Pinecone
```

### Step 8: Test (5 min)
```
Health Check:
curl https://vidzai-api.onrender.com/api/health

Query Test (in browser console):
fetch('https://vidzai-api.onrender.com/api/query', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({question: 'Who worked in Houston?'})
})
.then(r => r.json())
.then(d => console.log(d))

Frontend Test:
Open https://xxxxx.vercel.app → type question → get answer
```

---

## Environment Variables Matrix

| Variable | From | Goes To | Example |
|----------|------|---------|---------|
| `NEO4J_URI` | Neo4j Aura | Render Env | `neo4j+s://aba123.databases.neo4j.io` |
| `NEO4J_USER` | Neo4j Aura | Render Env | `neo4j` |
| `NEO4J_PASSWORD` | Neo4j Aura email | Render Env | `aB3xYz9qPw` |
| `PINECONE_API_KEY` | Pinecone Dashboard | Render Env | `abcd123-xyz789` |
| `PINECONE_INDEX` | You choose | Render Env | `email-knowledge-graph` |
| `LLAMA_API_KEY` | Groq/OpenAI/Ollama | Render Env | `gsk_abc123xyz` |
| `VITE_API_URL` | Your Render URL | Vercel Env | `https://vidzai-api.onrender.com` |

---

## Data Flow Diagram

```
STEP 1-4: Get Credentials
━━━━━━━━━━━━━━━━━━━━━━━━

Neo4j Aura     →  NEO4J_URI, NEO4J_PASSWORD
Pinecone       →  PINECONE_API_KEY
Groq/OpenAI    →  LLAMA_API_KEY


STEP 5: Backend
━━━━━━━━━━━━━━

Render (Backend)
  ├─ Gets: NEO4J_*, PINECONE_*, LLAMA_* from environment variables
  ├─ Runs: Flask API at https://vidzai-api.onrender.com
  └─ Purpose: Connects everything together


STEP 6: Frontend
━━━━━━━━━━━━━━━

Vercel (Frontend)
  ├─ Gets: VITE_API_URL (points to Render)
  ├─ Runs: React app at https://xxxxx.vercel.app
  └─ Purpose: User interface


STEP 7: Populate (LOCAL only)
━━━━━━━━━━━━━━━━━━━━━━━━━━━

Milestone-2.py → Reads emails → Extracts entities → Writes to Neo4j Aura
Milestone-3.py → Reads Neo4j → Creates embeddings → Writes to Pinecone


STEP 8: User Query
━━━━━━━━━━━━━━━━━

1. User opens frontend: https://xxxxx.vercel.app
2. Types question: "Who worked in Houston?"
3. Frontend sends: POST to https://vidzai-api.onrender.com/api/query
4. Backend queries:
   ├─ Neo4j Aura: "Find Person nodes in Houston"
   ├─ Pinecone: "Find emails about Houston"
   └─ LLM API: "Generate answer from context"
5. Backend returns combined answer
6. Frontend displays with evidence
```

---

## Testing Checklist

```
☐ PHASE 1 COMPLETE
  ☐ Neo4j Aura instance created and accessible
  ☐ Pinecone index created: email-knowledge-graph
  ☐ LLM API key obtained
  ☐ All credentials saved

☐ PHASE 2 COMPLETE
  ☐ Render service deployed
  ☐ All environment variables added to Render
  ☐ Backend health check: curl returns {"status": "ok"}
  ☐ Backend URL saved: https://vidzai-api.onrender.com

☐ PHASE 3 COMPLETE
  ☐ Vercel deployment finished
  ☐ VITE_API_URL set to Render URL
  ☐ Frontend loads without errors
  ☐ Frontend URL saved: https://xxxxx.vercel.app

☐ PHASE 4 COMPLETE
  ☐ Milestone-2.py ran successfully
  ☐ Neo4j has entities (verify in Neo4j Browser)
  ☐ Milestone-3.py ran successfully
  ☐ Pinecone has vectors (verify in Pinecone dashboard)

☐ PHASE 5 COMPLETE
  ☐ Health check passes
  ☐ Query endpoint returns answer
  ☐ Frontend query works end-to-end
  ☐ No CORS errors in console
```

---

## Troubleshooting: What to Check When X Fails

**Frontend won't load**
→ Check: Browser console for errors
→ Check: Vercel deployment logs
→ Fix: Redeploy in Vercel

**Backend 502 error**
→ Check: Render service is running
→ Wait: Render free tier cold starts take 30s
→ Check: Render logs for errors

**"Cannot connect to Neo4j"**
→ Check: NEO4J_URI format (should start with neo4j+s://)
→ Check: NEO4J_PASSWORD is correct (copy from email)
→ Check: Neo4j instance is running in Aura

**"Pinecone API error"**
→ Check: PINECONE_API_KEY is correct
→ Check: Index name is "email-knowledge-graph"
→ Check: Pinecone quota not exceeded

**No results from query**
→ Check: Milestone-2.py completed successfully
→ Check: Milestone-3.py completed successfully
→ Check: Neo4j has data (query: MATCH (n) RETURN COUNT(n))
→ Check: Pinecone has vectors (check dashboard stats)

**"CORS error" in frontend**
→ Check: Backend CORS is configured (it is)
→ Fix: Restart Render service

---

## URLs You'll Need

```
GitHub Repo:           https://github.com/Dev2141/deployment
Render Dashboard:      https://dashboard.render.com
Vercel Dashboard:      https://vercel.com/dashboard
Neo4j Aura:            https://neo4j.com/cloud/aura
Pinecone:              https://www.pinecone.io

During Deployment:
├─ Your Render URL:    https://vidzai-api.onrender.com
├─ Your Vercel URL:    https://xxxxx.vercel.app
├─ Neo4j Browser:      https://neo4j.com/cloud/aura → Open Neo4j Browser
└─ Pinecone Dashboard: https://app.pinecone.io
```

---

## Common Mistakes to Avoid

❌ **DON'T**: Commit `.env` file to git
✅ **DO**: Use environment variables only

❌ **DON'T**: Skip running Milestone-2 & Milestone-3
✅ **DO**: Run both locally to populate databases

❌ **DON'T**: Use wrong format for NEO4J_URI
✅ **DO**: Use: neo4j+s://xxxx.databases.neo4j.io

❌ **DON'T**: Forget to set VITE_API_URL in Vercel
✅ **DO**: Add it as environment variable

❌ **DON'T**: Copy-paste passwords with quotes
✅ **DO**: Copy the actual value only

❌ **DON'T**: Test before populating databases
✅ **DO**: Run Milestones first, then test

---

## Files Checklist

In `/c/Users/padha/Desktop/deployment/`:

```
✅ frontend/                    React app
   ├─ src/api.ts               Updated with VITE_API_URL
   ├─ package.json
   └─ vite.config.ts

✅ api.py                       Flask backend
   ├─ CORS configured
   ├─ Health endpoint ready
   └─ No changes needed

✅ requirements.txt             Dependencies
   ├─ flask
   ├─ flask-cors
   ├─ gunicorn
   └─ (all others)

✅ COMPLETE_GUIDE.md            You are here
✅ DEPLOYMENT.md                Detailed steps
✅ CHECKLIST.md                 Interactive checklist
✅ CONFIG_REFERENCE.md          Config templates
✅ .env.example                 Environment template

✅ .gitignore                   Prevents committing secrets
```

---

## One Command to Test Backend

```bash
curl -X POST https://vidzai-api.onrender.com/api/query \
  -H "Content-Type: application/json" \
  -d '{"question": "Who worked in Houston?"}'
```

Expected response: JSON with answer

---

**💡 You now have everything needed to connect and deploy!**

**Start order:**
1. PHASE 1: Create accounts (30 min)
2. PHASE 2: Deploy backend (10 min)
3. PHASE 3: Deploy frontend (5 min)
4. PHASE 4: Populate data (20 min - LOCAL)
5. PHASE 5: Test (5 min)

**Total: ~1.5 hours first time**

**See COMPLETE_GUIDE.md for detailed instructions 📖**
