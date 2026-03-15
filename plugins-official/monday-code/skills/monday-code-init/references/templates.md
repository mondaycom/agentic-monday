# monday-code-init: Code Templates

This file contains the full code templates referenced by SKILL.md.
Copy these files verbatim when scaffolding a new monday.com code app.

---

## Frontend Templates

### frontend/package.json

```json
{
  "name": "{{APP_NAME}}-frontend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "rm -rf dist && tsc && vite build",
    "preview": "vite preview",
    "deploy": "mapps code:push -c -d dist -a ${MONDAY_APP_ID:?} --force && rm -f dist.zip"
  },
  "dependencies": {
    "@vibe/core": "^3.85.1",
    "monday-sdk-js": "^0.5.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@vitejs/plugin-react": "^4.2.0",
    "typescript": "^5.3.0",
    "vite": "^6.4.1"
  }
}
```

### frontend/vite.config.ts

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  define: {
    global: "globalThis",
  },
  build: {
    outDir: "dist",
  },
  server: {
    port: 3000,
  },
});
```

### frontend/index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{APP_NAME}}</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### frontend/index.js (monday-code CDN entry point)

```javascript
// This file is required by monday-code for CDN deployments
// It serves the built static files
```

### frontend/src/contexts/MondayContext.tsx

Critical monday SDK integration pattern. Handles both production (inside monday.com iframe) and local development modes.

```tsx
import React, { createContext, useContext, useEffect, useState } from "react";
import mondaySdk from "monday-sdk-js";

const monday = mondaySdk();

interface MondayContextType {
  monday: ReturnType<typeof mondaySdk>;
  userId: string;
  accountId: string;
  userName: string;
  theme: string;
  isLoading: boolean;
}

const MondayContext = createContext<MondayContextType | null>(null);

// Detect local development (not inside monday.com iframe)
const isLocalDev =
  typeof window !== "undefined" &&
  !window.location.ancestorOrigins?.length &&
  import.meta.env.DEV;

export function MondayProvider({ children }: { children: React.ReactNode }) {
  const [userId, setUserId] = useState("");
  const [accountId, setAccountId] = useState("");
  const [userName, setUserName] = useState("");
  const [theme, setTheme] = useState("light");
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    if (isLocalDev) {
      // Mock data for local development
      setUserId("test-user-1");
      setAccountId("test-account-1");
      setUserName("Local Developer");
      setTheme("light");
      setIsLoading(false);
      return;
    }

    monday.get("context").then((res: any) => {
      const ctx = res.data;
      setUserId(String(ctx.user?.id || ""));
      setAccountId(String(ctx.account?.id || ""));
      setUserName(ctx.user?.name || "");
      setTheme(ctx.theme || "light");
      setIsLoading(false);
    });

    monday.listen("context", (res: any) => {
      if (res.data?.theme) setTheme(res.data.theme);
    });
  }, []);

  return (
    <MondayContext.Provider
      value={{ monday, userId, accountId, userName, theme, isLoading }}
    >
      {children}
    </MondayContext.Provider>
  );
}

export function useMondayContext() {
  const ctx = useContext(MondayContext);
  if (!ctx) throw new Error("useMondayContext must be used within MondayProvider");
  return ctx;
}
```

### frontend/src/services/api.ts

```typescript
import mondaySdk from "monday-sdk-js";

const monday = mondaySdk();

const isLocalDev =
  !window.location.ancestorOrigins?.length &&
  import.meta.env.DEV;

async function getSessionToken(): Promise<string> {
  if (isLocalDev) {
    return import.meta.env.VITE_DEV_TOKEN || "dev-token";
  }

  const token = await monday.get("sessionToken");
  return token.data;
}

async function getBackendUrl(): Promise<string> {
  if (isLocalDev) {
    return "http://localhost:8080";
  }
  const response = await monday.get("context");
  // @ts-ignore
  return response.data.appVersion.mondayCodeHostingUrl;
}

export async function apiFetch(path: string, options: RequestInit = {}) {
  const token = await getSessionToken();
  const backendUrl = await getBackendUrl();
  const res = await fetch(`${backendUrl}${path}`, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      Authorization: token,
      ...options.headers,
    },
  });
  if (!res.ok) throw new Error(`API error: ${res.status}`);
  return res.json();
}
```

### frontend/src/App.tsx

```tsx
import React from "react";
import { useMondayContext } from "./contexts/MondayContext";

export default function App() {
  const { userName, isLoading, theme } = useMondayContext();

  if (isLoading) return <div>Loading...</div>;

  return (
    <div data-theme={theme}>
      <h1>Welcome, {userName}</h1>
    </div>
  );
}
```

### frontend/src/main.tsx

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import { MondayProvider } from "./contexts/MondayContext";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <MondayProvider>
      <App />
    </MondayProvider>
  </React.StrictMode>
);
```

### frontend/.env.example

```bash
# For local development only - generate a JWT with test user data
VITE_DEV_TOKEN=your-dev-jwt-token
```

---

## Backend Templates

### backend/package.json

```json
{
  "name": "{{APP_NAME}}-backend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "build": "rm -rf dist && tsc",
    "dev": "tsx watch --require ./preload.cjs src/server.ts",
    "start": "node --require ./preload.cjs index.js",
    "deploy": "mapps code:push -a ${MONDAY_APP_ID:?} --force && rm -f code.tar.gz"
  },
  "dependencies": {
    "@mondaycom/apps-sdk": "^3",
    "cors": "^2.8.5",
    "dotenv": "^16.4.0",
    "express": "^4.18.0",
    "jsonwebtoken": "^9.0.0",
    "mongodb": "^6.3.0",
    "zod": "^3.22.0"
  },
  "devDependencies": {
    "@types/cors": "^2.8.0",
    "@types/express": "^4.17.0",
    "@types/jsonwebtoken": "^9.0.0",
    "tsx": "^4.7.0",
    "typescript": "^5.3.0"
  }
}
```

### backend/preload.cjs

```javascript
require('dotenv').config();
```

### backend/index.js (monday-code serverless entry point)

```javascript
import path from "path";
import { fileURLToPath } from "url";

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const { default: app } = await import(path.join(__dirname, "dist", "app.js"));

const PORT = getSecret("PORT") || 8080;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### backend/src/app.ts

```typescript
import express from "express";
import cors from "cors";
import { authMiddleware } from "./middleware/auth.js";
import { healthRouter } from "./routes/health.js";

const app = express();

app.use(cors());
app.use(express.json({ limit: "5mb" }));

// Public routes (no auth)
app.use("/health", healthRouter);

// Protected routes (require monday JWT)
app.use("/api", authMiddleware);

// Add your API routes here:
// app.use("/api/items", itemsRouter); # items router is a standard express controller

export default app;
```

### backend/src/routes/health.ts

```typescript
import { Router } from "express";

export const healthRouter = Router();

healthRouter.get("/", (_req, res) => {
  res.json({ status: "healthy", timestamp: new Date().toISOString() });
});
```

### backend/src/middleware/auth.ts (fullstack / session tokens)

JWT authentication using monday session tokens:

```typescript
import jwt from "jsonwebtoken";
import type { Request, Response, NextFunction } from "express";

export interface AuthContext {
  userId: string;
  accountId: string;
}

declare global {
  namespace Express {
    interface Request {
      auth?: AuthContext;
    }
  }
}

export function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith("Bearer ")) {
    return res.status(401).json({ error: "Missing authorization token" });
  }

  const token = authHeader.split(" ")[1];
  const secret = getSecret("MONDAY_CLIENT_SECRET");

  if (!secret) {
    console.error("MONDAY_CLIENT_SECRET not configured");
    return res.status(500).json({ error: "Server misconfigured" });
  }

  try {
    const decoded = jwt.verify(token, secret) as any;

    // monday.com tokens use two formats:
    // Production: { dat: { user_id, account_id, client_id } }
    // Development: { userId, accountId }
    if (decoded.dat) {
      req.auth = {
        userId: String(decoded.dat.user_id),
        accountId: String(decoded.dat.account_id),
      };
    } else {
      req.auth = {
        userId: String(decoded.userId),
        accountId: String(decoded.accountId),
      };
    }

    next();
  } catch (err) {
    return res.status(401).json({ error: "Invalid token" });
  }
}
```

### backend/src/middleware/auth.ts (automation webhook variant)

JWT authentication using automation tokens:

```typescript
import jwt from "jsonwebtoken";
import type { Request, Response, NextFunction } from "express";

/** Define the session property on the request object   */
declare global {
  namespace Express {
    interface Request {
      session: {
        accountId: string;
        userId: string;
        backToUrl: string | undefined;
        shortLivedToken: string | undefined;
      };
    }
  }
}

export default async function authenticationMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
) {
  try {
    const authorization = req.headers.authorization ?? req.query?.token;

    if (typeof authorization !== "string") {
      res
        .status(401)
        .json({ error: "not authenticated, no credentials in request" });
      return;
    }

    if (typeof process.env.MONDAY_SIGNING_SECRET !== "string") {
      res.status(500).json({ error: "Missing MONDAY_SIGNING_SECRET (should be in .env file)" });
      return;
    }

    const { accountId, userId, backToUrl, shortLivedToken } = jwt.verify(
      authorization,
      getSecret("MONDAY_SIGNING_SECRET")
    ) as any;

    req.session = { accountId, userId, backToUrl, shortLivedToken };

    // shortLivedToken is only included for automations, not for regular session tokens.
    // If present, use it to make authenticated GraphQL requests back to monday.com
    // on behalf of the user who triggered the automation.

    next();
  } catch (err) {
    res
      .status(401)
      .json({ error: "authentication error, could not verify credentials" });
  }
}
```

### backend/src/db/connection.ts (Document DB / MongoDB)

```typescript
import { MongoClient, Db } from "mongodb";

let client: MongoClient | null = null;
let db: Db | null = null;

export async function getDb(): Promise<Db> {
  if (db) return db;

  // monday-code auto-injects MNDY_MONGODB_CONNECTION_STRING after first deploy
  const uri = getSecret("MNDY_MONGODB_CONNECTION_STRING");
  if (!uri) {
    throw new Error(
      "MNDY_MONGODB_CONNECTION_STRING not set. " +
      "For local dev: use docker run -d -p 27017:27017 mongo:7 and set the env var. " +
      "For production: deploy once to monday-code and the var is auto-injected."
    );
  }

  client = new MongoClient(uri);
  await client.connect();
  db = client.db();
  return db;
}
```

### backend/src/utils/secrets.ts (Secrets Manager)

```typescript
import { SecretsManager } from "@mondaycom/apps-sdk";

const secretsManager = new SecretsManager();

export async function getSecret(key: string): Promise<string | undefined> {
  try {
    const { value } = await secretsManager.getSecret(key);
    return value;
  } catch {
    // Falls back to env var in local dev
    return process.env[key];
  }
}
```

### backend/src/utils/env-vars.ts (Environment Variables)

```typescript
import { EnvironmentVariablesManager } from "@mondaycom/apps-sdk";

const envManager = new EnvironmentVariablesManager();

export function getEnvVar(key: string): string {
  const value = envManager.getEnvVar(key);
  if (!value) {
    throw new Error(`Environment variable not set: ${key}`);
  }
  return value;
}
```

### backend/src/utils/queue.ts (Optional: async queue processing)

Use this when the user wants to implement async processing with monday-code's built-in queue system:

```typescript
import { Logger, Queue, EnvironmentVariablesManager } from "@mondaycom/apps-sdk";

const queue = new Queue();
const logTag = "QueueService";
const logger = new Logger(logTag);

export const produceMessage = async (message) => {
    logger.info(`produce message received ${message}`);
    const messageId = await queue.publishMessage(message);
    logger.info(`Message ${messageId} published.`);
    return messageId;
}

export const readQueueMessage = ({ body, query }) => {
    const envMessageSecret = getSecret("MNDY_TOPIC_MESSAGES_SECRET");
    logger.info(`expected queue secret value: ${envMessageSecret}`)
    logger.info(`queue message received body ${JSON.stringify(body)}`)
    logger.info(`queue message query params ${JSON.stringify(query)}`)
    if (!queue.validateMessageSecret(query.secret))  {
        logger.info("Queue message received is not valid, since secret is not matched, this message could come from an attacker.");
        throw new Error('not allowed');
    }
    logger.info("Queue message received successfully.");
    // process the queue message payload...
};
```

### backend/.env.example

```bash
# monday-code auto-injects this after first deploy
MNDY_MONGODB_CONNECTION_STRING=mongodb://localhost:27017/{{APP_NAME}}

MONDAY_CLIENT_SECRET=your-client-secret

# Optional
PORT=8080
```
