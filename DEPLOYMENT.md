# 🚀 Deployment Guide: Vidzai to Vercel + Render

This repository contains the production-ready code for deploying the Vidzai AI Knowledge Graph project to **Vercel** (Frontend) and **Render** (Backend).

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Part 1: Vercel Frontend Deployment](#part-1-vercel-frontend-deployment)
- [Part 2: Render Backend Deployment](#part-2-render-backend-deployment)
- [Part 3: External Services Setup](#part-3-external-services-setup)
- [Part 4: Integration & Testing](#part-4-integration--testing)
- [Troubleshooting](#troubleshooting)

---

## Overview

Vidzai: AI-Based Knowledge Graph & Hybrid RAG is deployed across:

- **Frontend**: React + Vite → **Vercel**
- **Backend**: Flask API → **Render**
- **Databases**: Neo4j Aura + Pinecone (called via API)
- **LLM**: Ollama or external LLM service

All deployments use free tiers for minimal cost.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Browser → VERCEL (React Frontend)                                       │
│                ↓ HTTPS calls                                            │
│          RENDER (Flask Backend)                                         │
│                ↓                                                         │
│    ┌───────────┼──────────────┬────────────────────┐                   │
│    ↓           ↓              ↓                    ↓                    │
│ Neo4j Aura  Pinecone     Ollama/LLM         GitHub Logs                │
│(Graph DB)  (Vector DB)  (LLM Generation)   (Monitoring)               │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- GitHub account with this repository pushed ✓
- Vercel account (free)
- Render account (free)
- Neo4j Aura account (free)
- Pinecone account (free tier)
- LLM API key (Groq, OpenAI, or local Ollama)

---

## Part 1: Vercel Frontend Deployment

### Step 1.1: Create Vercel Account

1. Go to https://vercel.com
2. Click "Sign Up"
3. Select "GitHub" and authorize
4. Allow Vercel to access your GitHub repositories

### Step 1.2: Import Project

1. Click "New Project" or "Add New" → "Project"
2. Search for: `deployment`
3. Click "Import"

### Step 1.3: Configure Build Settings

Vercel should auto-detect these settings:

| Setting | Value |
|---------|-------|
| Framework | Vite |
| Build Command | `npm install && npm run build` |
| Output Directory | `frontend/dist` |
| Root Directory | `frontend` |

**If not auto-detected**, set manually in Project Settings.

### Step 1.4: Set Environment Variables

In Vercel Dashboard:

1. Go to **Settings** → **Environment Variables**
2. Add variable:
   ```
   VITE_API_URL = https://vidzai-api.onrender.com
   ```
   *(Update with Render URL after deploying backend)*

### Step 1.5: Deploy

1. Click **Deploy**
2. Wait for build to complete (~2-3 minutes)
3. Copy your Vercel URL: `https://your-project.vercel.app`

✅ **Frontend is live!**

---

## Part 2: Render Backend Deployment

### Step 2.1: Create Render Account

1. Go to https://render.com
2. Click "Get Started"
3. Sign up with GitHub
4. Authorize Render

### Step 2.2: Create Web Service

1. Click **New** → **Web Service**
2. Search for: `deployment`
3. Connect the repository

### Step 2.3: Configure Service

Fill in the following:

| Field | Value |
|-------|-------|
| **Name** | `vidzai-api` |
| **Environment** | `Python 3` |
| **Build Command** | `pip install -r requirements.txt` |
| **Start Command** | `gunicorn api:app` |
| **Region** | *Choose closest to you* |

### Step 2.4: Set Environment Variables

In Render Dashboard → **Environment**:

Add these variables (get values from Part 3):

```
NEO4J_URI=neo4j+s://[YOUR_AURA_URI]
NEO4J_USER=neo4j
NEO4J_PASSWORD=[YOUR_AURA_PASSWORD]

PINECONE_API_KEY=[YOUR_PINECONE_KEY]
PINECONE_INDEX=email-knowledge-graph
PINECONE_ENV=prod

LLAMA_API_KEY=[YOUR_LLM_KEY]
MODEL_NAME=gpt-oss:120b-cloud

FLASK_ENV=production
FLASK_DEBUG=false
```

### Step 2.5: Deploy

1. Click **Create Web Service**
2. Render builds and deploys automatically
3. Wait for deployment to complete (~5-10 minutes)
4. Copy your Render URL: `https://vidzai-api.onrender.com`

✅ **Backend is live!**

### Step 2.6: Update Vercel with Backend URL

Now update Vercel with the Render URL:

1. Go back to Vercel Dashboard
2. **Settings** → **Environment Variables**
3. Update `VITE_API_URL`:
   ```
   VITE_API_URL = https://vidzai-api.onrender.com
   ```
4. Click **Save**
5. Redeployment triggers automatically

---

## Part 3: External Services Setup

### Step 3.1: Neo4j Aura Setup

1. Go to https://neo4j.com/cloud/aura
2. Click **Create**
3. Choose **Free Tier**
4. Select region (closest to you)
5. Create instance
6. **Copy connection details**:
   - `URI`: neo4j+s://xxxxxx.databases.neo4j.io
   - `Username`: neo4j
   - `Password`: [auto-generated, saved in your email]

7. Add to Render environment variables

### Step 3.2: Pinecone Setup

1. Go to https://www.pinecone.io
2. Click **Sign in** → **GitHub**
3. Authorize
4. **Create index**:
   - Name: `email-knowledge-graph`
   - Dimension: `1024` (for multilingual-e5-large)
   - Metric: `cosine`

5. Copy **API Key** from dashboard
6. Add to Render environment

### Step 3.3: LLM Setup (Choose One)

#### Option A: Groq (Recommended - Free)

1. Go to https://console.groq.com
2. Sign up with GitHub
3. Copy API key
4. Set in Render: `LLAMA_API_KEY=[YOUR_GROQ_KEY]`

#### Option B: OpenAI (Paid, but free credits available)

1. Go to https://platform.openai.com
2. Sign up
3. Add payment method (free credits provided)
4. Create API key
5. Set in Render: `OPENAI_API_KEY=[YOUR_KEY]`

#### Option C: Ollama (Local, Free)

1. Install Ollama: https://ollama.ai
2. Run locally or on separate server
3. Set in Render: `OLLAMA_BASE_URL=http://your-ollama-server:11434`

---

## Part 4: Integration & Testing

### Step 4.1: Verify Backend Health

Test your backend is running:

```bash
curl https://vidzai-api.onrender.com/api/health
```

Expected response:
```json
{
  "status": "ok",
  "missing_env": [],
  "model": "gpt-oss:120b-cloud",
  "pinecone_index": "email-knowledge-graph"
}
```

### Step 4.2: Verify Frontend Loads

1. Open https://your-project.vercel.app
2. React app should load
3. Open browser DevTools → Console
4. Should see no CORS errors

### Step 4.3: Test API Connection

In browser console, test:

```javascript
fetch('https://vidzai-api.onrender.com/api/health')
  .then(r => r.json())
  .then(d => console.log('Success:', d))
  .catch(e => console.error('Error:', e))
```

### Step 4.4: Test Full Query

1. Go to frontend
2. Enter sample question: "Who communicated about natural gas?"
3. Should get response from backend
4. Check network tab for API calls working

---

## Troubleshooting

### Issue: CORS Errors

**Solution**: Backend CORS is already configured. If still getting errors:

1. Check Render environment variables are set
2. Verify backend starts without errors in Render logs
3. Bounce backend service (Render dashboard → Service → Restart)

### Issue: Backend 502 Bad Gateway

**Solution**: Cold start on Render free tier

1. Backend goes to sleep after 15 minutes of inactivity
2. First request takes 30+ seconds
3. Normal behavior - will work fine after initial request

### Issue: Neo4j Connection Failed

**Solution**:

1. Verify NEO4J_URI format: should start with `neo4j+s://`
2. Check password in environment variable (copy-paste from email)
3. Verify Neo4j Aura instance is running in dashboard

### Issue: Pinecone Vector Search Returns Empty

**Solution**:

1. Check PINECONE_API_KEY is correct
2. Verify index name matches (`email-knowledge-graph`)
3. You need to populate Pinecone:
   - Run locally: `python Milestone-3.py` to index embeddings
   - Or use GitHub Actions for scheduled indexing

### Issue: Frontend Shows Blank Page

**Solution**:

1. Open browser DevTools → Console tab
2. Check for errors
3. Verify `VITE_API_URL` is set in Vercel
4. Redeploy frontend (Vercel → Deployments → Redeploy)

---

## Maintenance

### Auto-Redeployment on Git Push

Both Vercel and Render automatically rebuild and redeploy when you push to GitHub's main branch.

### Render Cold Starts

Render free tier sleeps after 15 minutes. To keep alive:

1. Add cron job that pings `/api/health` every 10 minutes
2. Or upgrade to paid tier ($7/month)

### Scaling

If you hit limits:

| Service | Free Limit | Upgrade Price |
|---------|-----------|---------------|
| Vercel | Unlimited bandwidth | - |
| Render | 750 hours/month | $7/month |
| Neo4j Aura | 1 GB storage | Pay per GB |
| Pinecone | 100K vectors | Pay per vector |

---

## Quick Reference

| Aspect | Service | URL |
|--------|---------|-----|
| Frontend | Vercel | https://your-project.vercel.app |
| Backend | Render | https://vidzai-api.onrender.com |
| Graph DB | Neo4j Aura | Managed service |
| Vector DB | Pinecone | Managed service |
| GitHub | Dev2141 | https://github.com/Dev2141/deployment |

---

## Support

For issues, check:

1. **Render Logs**: Dashboard → Service → Logs
2. **Vercel Logs**: Dashboard → Deployments → Logs
3. **Network Tab**: Browser DevTools → Network
4. **Console Errors**: Browser DevTools → Console

---

**Happy deploying! 🚀**
