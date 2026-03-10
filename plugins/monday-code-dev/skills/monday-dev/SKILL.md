---
name: monday-dev
description: Start development servers and local environment for monday code apps
argument-hint: "[frontend|backend|all]"
user-invocable: true
allowed-tools: ["Bash", "Read", "Write", "Glob", "Grep"]
---

# monday-dev

Start development servers and set up the local environment for monday code apps.

## When to Use

- User wants to start the development server
- User wants to run frontend dev server (Vite)
- User wants to run backend dev server (Express with watch mode)
- User wants to run both frontend and backend
- User wants to set up local MongoDB for Document DB

## Instructions

### Step 1: Detect Project Structure

Check which directories exist:
```bash
ls -d frontend/ backend/ 2>/dev/null
```

- `frontend/` exists → Frontend app
- `backend/` exists → Backend app
- Both exist → Fullstack app

### Step 2: Pre-Flight Checks

**Check dependencies are installed:**
```bash
# Frontend
[ -d "frontend/node_modules" ] || (cd frontend && npm install)

# Backend
[ -d "backend/node_modules" ] || (cd backend && npm install)
```

**Check environment files:**

For backend, verify `.env` exists. If not, check for `.env.example`:
```bash
[ -f "backend/.env" ] || echo "WARNING: No .env file found in backend/"
```

Required backend env vars:
- `MONDAY_CLIENT_SECRET` - For JWT verification (set in Developer Center > OAuth)
- `MNDY_MONGODB_CONNECTION_STRING` - For Document DB (local: `mongodb://localhost:27017/appname`)

### Step 3: Start Local MongoDB (if backend uses Document DB)

Check if the backend uses MongoDB:
```bash
grep -r "MNDY_MONGODB_CONNECTION_STRING\|mongodb\|MongoClient" backend/src/ 2>/dev/null
```

If it does, ensure a local MongoDB is running:

```bash
# Check if MongoDB is already running
docker ps --filter name=monday-app-mongo --format "{{.Names}}" 2>/dev/null

# If not running, start one:
docker run -d --name monday-app-mongo -p 27017:27017 mongo:7
```

Set in `backend/.env`:
```
MNDY_MONGODB_CONNECTION_STRING=mongodb://localhost:27017/monday-app
```

### Step 4: Start Dev Servers

**Frontend only:**
```bash
cd frontend && npm run dev
```

Starts Vite on port 3000 with HMR. Access at http://localhost:3000.

**Backend only:**
```bash
cd backend && npm run dev
```

Starts Express with tsx watch mode on port 8080 (or `PORT` env var). Auto-reloads on file changes.

**Fullstack (both servers):**

Run each in a separate background process:
```bash
# Start backend first (frontend may depend on it)
cd backend && npm run dev &

# Then frontend
cd frontend && npm run dev
```

Or suggest the user to open two terminal tabs.

### Step 5: Local Dev Detection

The frontend MondayContext should auto-detect local development. It works by checking:

```typescript
const isLocalDev =
  !window.location.ancestorOrigins?.length &&  // Not inside monday.com iframe
  import.meta.env.DEV;                         // Vite dev mode
```

When in local dev mode:
- Monday SDK calls return mock data (test-user-1, test-account-1)
- API calls use `VITE_DEV_TOKEN` for authentication
- No real monday.com connection needed

### Step 6: Testing with monday.com
To test the app inside monday.com, you will need to create the app and the desired app feature in the monday.com Developer Center.
You can use the local dev server for the frontend and a tunnel for the backend.


If building a backend for automations/integrations on monday.com production with real monday.com webhooks/integrations, the user needs a tunnel:

```bash
# Using ngrok (or similar)
mapps tunnel:create -p 8080
```

Then set the tunnel URL in monday.com Developer Center go to:
Your app > Features > your app feature > Deployment > External hosting

And put in the fielsd the tunnel URL (e.g. `https://548f50475f79.apps-tunnel.monday.app`)

For frontend testing inside monday.com:
- Use the local dev server URL (http://localhost:3000) as the feature's Deployment URL in Developer Center
