# Monday Code Test

Run Playwright tests for monday code apps with monday.com-specific helpers.

## Installation

```bash
/plugin install monday-code-test@agentic-monday-apps-framework
```

## Usage

```bash
/monday-test             # Run all tests
/monday-test api         # API tests only
/monday-test ui          # UI tests only
/monday-test e2e         # E2E tests only
```

## What It Does

1. Detects existing test setup (or creates one)
2. Runs Playwright tests with project-based organization
3. Provides monday.com-specific test helpers:
   - JWT token generation matching monday.com's `{ dat: { user_id, account_id } }` format
   - Multi-tenant isolation testing patterns
   - Monday SDK mocking for UI tests
   - Document DB cleanup helpers

## Test Helpers

### JWT Auth
```typescript
import { authHeaders } from "./helpers/auth";
const response = await request.get("/api/items", { headers: authHeaders() });
```

### Multi-Tenant Isolation
```typescript
const headersA = authHeaders("user-a", "account-a");
const headersB = authHeaders("user-b", "account-b");
// Verify account-a data is invisible to account-b
```

## Requirements

- Dev servers running (`/monday-dev`)
- Playwright installed
- `MONDAY_CLIENT_SECRET` set (for JWT generation)
