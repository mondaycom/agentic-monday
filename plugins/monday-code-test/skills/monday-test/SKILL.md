---
name: monday-test
description: Run Playwright tests (API/UI/E2E) for monday code apps with JWT auth helpers
argument-hint: "[api|ui|e2e|all]"
user-invocable: true
allowed-tools: ["Bash", "Read", "Write", "Glob", "Grep"]
---

# monday-test

Run and manage Playwright tests for monday code apps.

## When to Use

- User wants to run all tests or a specific test suite
- User wants to set up testing infrastructure
- User wants to debug test failures
- User wants to create new tests

## Instructions

### Step 1: Detect Test Setup

Check if tests are configured:
```bash
ls tests/package.json tests/playwright.config.ts 2>/dev/null
```

If no test setup exists, offer to create one (see "Setting Up Tests" below).

### Step 2: Run Tests

**Run all tests:**
```bash
cd tests && npm test
```

**Run by project (argument-based):**
```bash
# API tests only
cd tests && npx playwright test --project=api

# UI tests only
cd tests && npx playwright test --project=ui

# E2E tests only
cd tests && npx playwright test --project=e2e
```

**Run specific test file:**
```bash
cd tests && npx playwright test tests/api/health.test.ts
```

**Debug mode:**
```bash
cd tests && PWDEBUG=1 npx playwright test tests/api/health.test.ts
```

### Step 3: Interpret Results

After tests run, check the output. If tests fail:

1. Read the error message carefully
2. For API tests: check if backend is running on the expected port
3. For UI tests: check if frontend is running and the selectors match
4. For auth failures: verify the test JWT token is correct

View HTML report:
```bash
cd tests && npx playwright show-report
```

## Setting Up Tests

If tests don't exist yet, create the following structure:

```
tests/
├── package.json
├── playwright.config.ts
├── helpers/
│   ├── auth.ts
│   ├── cleanup.ts
│   └── ui-setup.ts
├── api/
│   └── health.test.ts
└── ui/
    └── app.test.ts
```

**package.json:**
```json
{
  "name": "monday-app-tests",
  "type": "module",
  "scripts": {
    "test": "playwright test",
    "test:api": "playwright test --project=api",
    "test:ui": "playwright test --project=ui",
    "test:e2e": "playwright test --project=e2e",
    "report": "playwright show-report"
  },
  "devDependencies": {
    "@playwright/test": "^1.48.0",
    "@types/jsonwebtoken": "^9.0.0",
    "jsonwebtoken": "^9.0.0",
    "typescript": "^5.3.0"
  }
}
```

**playwright.config.ts:**
```typescript
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: ".",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [["list"], ["json", { outputFile: "test-results.json" }]],

  projects: [
    {
      name: "api",
      testMatch: /api\/.*\.test\.ts/,
      use: {
        baseURL: process.env.BACKEND_URL || "http://localhost:8080",
      },
    },
    {
      name: "ui",
      testMatch: /ui\/.*\.test\.ts/,
      use: {
        ...devices["Desktop Chrome"],
        baseURL: process.env.FRONTEND_URL || "http://localhost:3000",
      },
    },
    {
      name: "e2e",
      testMatch: /e2e\/.*\.test\.ts/,
      use: {
        ...devices["Desktop Chrome"],
        baseURL: process.env.FRONTEND_URL || "http://localhost:3000",
      },
    },
  ],
});
```

**helpers/auth.ts** - JWT token generation matching monday.com's format:
```typescript
import jwt from "jsonwebtoken";

const TEST_SECRET = process.env.MONDAY_CLIENT_SECRET || "test-secret";

// Generate a token in monday.com production format
// monday.com tokens use: { dat: { user_id, account_id, client_id } }
export function generateTestToken(
  userId: string = "test-user-1",
  accountId: string = "test-account-1"
): string {
  return jwt.sign(
    {
      dat: {
        user_id: userId,
        account_id: accountId,
        client_id: "test-client",
      },
    },
    TEST_SECRET,
    { expiresIn: "1h" }
  );
}

// Generate auth headers for API requests
export function authHeaders(userId?: string, accountId?: string) {
  return {
    Authorization: `Bearer ${generateTestToken(userId, accountId)}`,
    "Content-Type": "application/json",
  };
}
```

**helpers/cleanup.ts** - Database cleanup between tests:
```typescript
import { MongoClient } from "mongodb";

const MONGO_URI =
  process.env.MNDY_MONGODB_CONNECTION_STRING ||
  "mongodb://localhost:27017/monday-app-test";

let client: MongoClient | null = null;

export async function getTestDb() {
  if (!client) {
    // Add mongodb to devDependencies if using this helper
    const { MongoClient: MC } = await import("mongodb");
    client = new MC(MONGO_URI);
    await client.connect();
  }
  return client.db();
}

export async function cleanupCollections(...names: string[]) {
  const db = await getTestDb();
  for (const name of names) {
    await db.collection(name).deleteMany({});
  }
}

export async function closeTestDb() {
  if (client) {
    await client.close();
    client = null;
  }
}
```

**api/health.test.ts** - Example API test:
```typescript
import { test, expect } from "@playwright/test";
import { authHeaders } from "../helpers/auth";

test.describe("Health Check", () => {
  test("returns healthy status", async ({ request }) => {
    const response = await request.get("/health");
    expect(response.status()).toBe(200);

    const body = await response.json();
    expect(body.status).toBe("healthy");
  });
});

test.describe("Authentication", () => {
  test("rejects requests without auth token", async ({ request }) => {
    const response = await request.get("/api/items");
    expect(response.status()).toBe(401);
  });

  test("accepts requests with valid auth token", async ({ request }) => {
    const response = await request.get("/api/items", {
      headers: authHeaders(),
    });
    // Should not be 401 (might be 200 or 404 depending on route)
    expect(response.status()).not.toBe(401);
  });
});
```

**ui/app.test.ts** - Example UI test:
```typescript
import { test, expect } from "@playwright/test";

test.describe("App UI", () => {
  test("loads the app", async ({ page }) => {
    await page.goto("/");
    // In local dev mode, MondayContext provides mock data
    await expect(page.locator("body")).toBeVisible();
  });
});
```

After creating, install dependencies:
```bash
cd tests && npm install && npx playwright install --with-deps chromium
```

## Common Testing Patterns for Monday Apps

**Multi-tenant test isolation:**
```typescript
// Each test uses a unique accountId to avoid cross-test contamination
test("user A sees only their data", async ({ request }) => {
  const headers = authHeaders("user-a", "account-a");
  // Create data as user A
  await request.post("/api/items", { headers, data: { title: "A's item" } });

  // Verify user B (different account) can't see it
  const headersB = authHeaders("user-b", "account-b");
  const res = await request.get("/api/items", { headers: headersB });
  const items = await res.json();
  expect(items).toHaveLength(0);
});
```

**Testing monday SDK integration (frontend):**
```typescript
test.beforeEach(async ({ page }) => {
  // Mock the monday SDK before page loads
  await page.addInitScript(() => {
    (window as any).monday = {
      get: (type: string) => {
        if (type === "context")
          return Promise.resolve({
            data: { user: { id: 1, name: "Test" }, account: { id: 1 } },
          });
        if (type === "sessionToken")
          return Promise.resolve({ data: "mock-token" });
        return Promise.resolve({});
      },
      listen: () => {},
      execute: () => Promise.resolve({}),
    };
  });
});
```

## Notes

- Always ensure dev servers are running before running tests
- Use separate test database (append `-test` to DB name)
- JWT test tokens must match the format the auth middleware expects
- monday.com production tokens use `{ dat: { user_id, account_id } }` format
- Clean up test data after each test suite
- Use `accountId` isolation for multi-tenant testing
