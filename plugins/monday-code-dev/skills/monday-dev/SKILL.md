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

Or suggest the user open two terminal tabs.

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

For testing the backend with real monday.com webhooks/integrations, the user needs a tunnel:

```bash
# Using ngrok (or similar)
ngrok http 8080
```

Then set the tunnel URL in monday.com Developer Center > Features > Build URL.

For frontend testing inside monday.com:
- Use `ngrok http 3000` or similar
- Set the tunnel URL as the feature's build URL in Developer Center

### Development Tips

**Frontend:**
- Access at http://localhost:3000
- HMR enabled - changes reflect immediately
- Use `@vibe/core` components for monday.com look and feel
- Check browser console for Monday SDK debug info

**Backend:**
- Access at http://localhost:8080
- Auto-restart on file changes (tsx watch)
- Test health endpoint: `curl http://localhost:8080/health`
- Use the Logger from `@mondaycom/apps-sdk` instead of `console.log`

**Ports:**
- Frontend: 3000 (Vite default)
- Backend: 8080 (monday-code default)
- MongoDB: 27017 (Docker)

### Common Issues

**Port already in use:**
```bash
lsof -ti:3000 | xargs kill -9   # Kill frontend
lsof -ti:8080 | xargs kill -9   # Kill backend
```

**MongoDB connection fails:**
```bash
# Check if Docker container is running
docker ps --filter name=monday-app-mongo

# Restart if needed
docker start monday-app-mongo
```

**Environment variables not loading:**
- Verify `.env` file exists in `backend/`
- Restart dev server after `.env` changes
- Check `preload.cjs` loads dotenv: `require('dotenv').config()`

**TypeScript errors during watch:**
```bash
cd backend && npx tsc --noEmit   # Type-check without building
cd frontend && npx tsc --noEmit
```

## Notes

- Use `run_in_background` for long-running dev servers
- Provide clear instructions on how to stop servers (Ctrl+C)
- Remind about MongoDB setup if backend uses Document DB
- Frontend works standalone for UI development (mock mode)
- Backend needs `MONDAY_CLIENT_SECRET` for JWT auth to work
