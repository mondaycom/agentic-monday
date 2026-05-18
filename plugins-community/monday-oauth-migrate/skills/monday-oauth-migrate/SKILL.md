---
name: monday-oauth-migrate
description: "Migrate an existing monday.com app from the old OAuth flow to the new OAuth 2.1 flow. Changes token endpoint from oauth2/token to oauth_ms/oauth/token, adds refresh token support, updates token storage, and optionally adds PKCE. ALWAYS use when someone mentions migrating monday OAuth, switching to the new OAuth flow, adding refresh tokens, updating the monday token endpoint, token expiration, or when code calls auth.monday.com/oauth2/token. Also trigger on OAuth deadline/deprecation announcements. Do NOT use for new OAuth from scratch, non-monday OAuth (Google, Slack, Spotify), client credentials flow, DCR, or general monday debugging."
allowed-tools: Read Bash(grep *) Bash(find *) Bash(git *) Bash(npm *) Bash(yarn *) Bash(pnpm *)
---

# Monday.com OAuth Migration Skill

You are migrating a monday.com app from the **old OAuth flow** to the **new OAuth 2.1 flow**. Follow these phases in order. Each phase has clear entry/exit criteria. Do not skip phases.

## Phase 0: Detection — Does This App Need Migration?

Search the entire codebase for these patterns:

```
grep -r "auth.monday.com/oauth2/authorize" . --include="*.ts" --include="*.js" --include="*.py" --include="*.rb" --include="*.go" --include="*.java" --include="*.php" --include="*.cs" --include="*.rs" --include="*.jsx" --include="*.tsx" --include="*.mjs" --include="*.cjs" -l
```

```
grep -r "auth.monday.com/oauth2/token" . --include="*.ts" --include="*.js" --include="*.py" --include="*.rb" --include="*.go" --include="*.java" --include="*.php" --include="*.cs" --include="*.rs" --include="*.jsx" --include="*.tsx" --include="*.mjs" --include="*.cjs" -l
```

Also search for partial patterns like `oauth2/authorize` and `oauth2/token` in case the base URL is constructed dynamically.

**If NEITHER pattern is found:**
Tell the developer:
> "I didn't find any OAuth flow usage in this codebase (no calls to `auth.monday.com/oauth2/authorize` or `auth.monday.com/oauth2/token`). This app doesn't appear to be using the OAuth flow yet. This skill is specifically designed for **migrating** existing OAuth integrations. If you'd like to implement OAuth from scratch instead, I can help, but the migration workflow may not fit perfectly — want to continue anyway?"

If they want to continue, proceed but note that some steps (like finding need extra guidance.

**If patterns ARE found:** proceed to Phase 1.

Before moving on, report what you found: which files contain OAuth references, and give a brief summary of the current OAuth implementation structure you see (authorization redirect, callback handler, token exchange, token storage).

---

## Phase 1: Enable OAuth in the Dev Center (Manual Step)

This step cannot be automated. Ask the developer to do the following manually:

> **You need to enable the new OAuth flow in the monday.com Developer Center:**
>
> 1. Go to your app in the [Developer Center](https://monday.com/developers/apps)
> 2. Navigate to the **"OAuth & Permissions"** tab
> 3. Create a **new draft version** of your app (the toggle is per-version)
> 4. Enable the **"New OAuth Flow"** toggle for the draft version
> 5. Make sure the draft version is set to **"Active for me"** so you can test it
>
> Your scopes and redirect URIs will be cloned from the previous version automatically — no need to reconfigure them.
>
> **Tell me when you've completed this step so I can start migrating your code.**

Wait for the developer to confirm before proceeding. Do NOT start code changes until they confirm.

---

## Phase 2: PKCE Decision

Use AskUserQuestion to ask:

**Question:** "Would you like to add PKCE (Proof Key for Code Exchange) to your OAuth flow? PKCE adds an extra layer of security that protects against authorization code interception attacks. It's recommended for all apps and required for DCR-registered clients."

**Options:**
- **Yes, add PKCE (Recommended)** — Adds code_verifier generation and code_challenge to the authorization flow
- **No, skip PKCE** — Keep the flow simple without PKCE

Remember their choice — it affects Phase 3 and Phase 4.

---

## Phase 3: Migrate the Authorization Request (only if PKCE was chosen)

If the developer chose **No PKCE** — skip this phase entirely. The authorization endpoint (`GET https://auth.monday.com/oauth2/authorize`) stays exactly the same.

If the developer chose **Yes PKCE**, find where the authorization URL is constructed and add PKCE parameters:

### What to add

1. **Generate a code verifier** — a cryptographically random string (43-128 characters):

JavaScript/TypeScript:
```javascript
const crypto = require('crypto');
const codeVerifier = crypto.randomBytes(32).toString('base64url');
```

Python:
```python
import secrets, hashlib, base64
code_verifier = secrets.token_urlsafe(32)
```

2. **Compute the code challenge** — SHA-256 hash of the verifier, base64url-encoded:

JavaScript/TypeScript:
```javascript
const codeChallenge = crypto.createHash('sha256').update(codeVerifier).digest('base64url');
```

Python:
```python
code_challenge = base64.urlsafe_b64encode(hashlib.sha256(code_verifier.encode()).digest()).rstrip(b'=').decode()
```

3. **Add ONLY the PKCE params to the authorization URL — do NOT touch existing parameters:**
```
&code_challenge=CODE_CHALLENGE&code_challenge_method=S256
```

**Important:** The app may already use `state`, `scope`, `redirect_uri`, or other query parameters in the authorization URL. Preserve ALL existing parameters exactly as they are. Only append `code_challenge` and `code_challenge_method`. Do not replace, rename, or restructure the URL construction — just add the two new params.

4. **Store the code_verifier** — it must be available when exchanging the code for tokens in the callback. Use whatever mechanism the app already uses to maintain state between the authorization redirect and the callback (session, database, cookie, etc.). If the app already passes a `state` parameter, you can key the verifier by that existing state value — do not generate a new `state` to replace it.

Adapt the code to match the language, framework, and patterns already used in the developer's codebase. Do not introduce new dependencies if the language's standard library already provides crypto primitives. Do not restructure or rename existing variables, routes, or functions — make the minimal additions needed.

---

## Phase 4: Migrate the Token Exchange

This is the core migration step. Find the callback endpoint that:
1. Receives the authorization `code` from monday.com
2. Sends it (along with `client_id` and `client_secret`) to exchange for tokens

### 4a. Change the token exchange endpoint

Find the call to the **old** endpoint:
```
POST https://auth.monday.com/oauth2/token
```

Change it to the **new** endpoint:
```
POST https://auth.monday.com/oauth_ms/oauth/token
```

The request body parameters remain the same: `client_id`, `client_secret`, `code`, and optionally `redirect_uri`.

**If PKCE was chosen:** also include `code_verifier` in the request body — this is the same verifier generated in Phase 3.

### 4b. Handle the new response format

The old flow returned only an `access_token` (which never expired).
The new flow returns **both** an `access_token` (which expires) and a `refresh_token`:

```json
{
  "access_token": "eyJhbGciOi...",
  "refresh_token": "eyJhbGciOi...",
  "token_type": "Bearer",
  "scope": "boards:read users:read"
}
```

### 4c. Update token storage

This is critical. Find where the old `access_token` was being saved (database, file, env variable, key-value store, etc.) and ensure:

1. The `refresh_token` is **also saved** alongside the access token
2. Both tokens are associated with the same user/account context
3. The storage mechanism can handle updating both tokens (they will change on every refresh)

Look for patterns like:
- Database columns (e.g., `access_token` column — add a `refresh_token` column)
- Object/model properties storing the token
- Key-value store entries
- Environment variables or config files

If the developer stores tokens in a database, you may need to add a migration or alter the schema. If they use a simpler storage mechanism, just ensure both tokens are persisted.

---

## Phase 5: Add Token Refresh Logic

This is entirely new functionality — the old flow had no token expiration, so there was no refresh mechanism.

### 5a. Create the refresh function

Create a function that refreshes an expired access token. It calls the same token endpoint but with different parameters:

**Endpoint:**
```
POST https://auth.monday.com/oauth_ms/oauth/token
```

**Request body:**
```json
{
  "client_id": "YOUR_CLIENT_ID",
  "client_secret": "YOUR_CLIENT_SECRET",
  "refresh_token": "CURRENT_REFRESH_TOKEN",
  "grant_type": "refresh_token"
}
```

**Response:**
```json
{
  "access_token": "new_access_token...",
  "refresh_token": "new_refresh_token...",
  "token_type": "Bearer",
  "scope": "boards:read users:read"
}
```

Each refresh returns a **new** refresh_token. Always replace both the access token AND the refresh token in storage.

### 5b. Integrate refresh into the app's API call flow

Find where the app uses the access token to call the monday.com API (typically via `Authorization: Bearer <token>` header or the monday SDK). Add logic to handle token expiration:

**Recommended pattern — proactive check before API calls:**
- Decode the JWT access token and check the `exp` (expiration) claim
- If the token is expired or about to expire (within ~60 seconds), refresh it first
- Then proceed with the API call using the new token

**Alternative pattern — reactive refresh on 401:**
- Make the API call
- If the response is `401 Unauthorized`, attempt to refresh the token
- Retry the original API call with the new token
- If refresh also fails, the user needs to re-authorize

Choose whichever pattern best fits the existing codebase architecture. If the app already has middleware, interceptors, or a central API client, hook into that.

### 5c. Handle refresh failures

If a refresh call fails (e.g., refresh token was revoked or expired), the user must re-authorize through the full OAuth flow again. Make sure the app handles this gracefully — redirect to the authorization URL or show an appropriate error.

---

## Phase 6: Token Revocation (Optional)

Use AskUserQuestion to ask:

**Question:** "Would you like to add a token revocation function? This lets your app revoke tokens when a user disconnects or uninstalls (e.g., cleanup on uninstall webhook)."

**Options:**
- **Yes, add revocation** — Adds a function to revoke access or refresh tokens
- **No, skip** — Skip revocation for now

If **Yes**, create a revocation function:

**Endpoint:**
```
POST https://auth.monday.com/oauth_ms/oauth/revoke
```

**Request body:**
```json
{
  "token": "TOKEN_TO_REVOKE",
  "client_id": "YOUR_CLIENT_ID",
  "client_secret": "YOUR_CLIENT_SECRET",
  "token_type_hint": "access_token"
}
```

- `token_type_hint` can be either `"access_token"` or `"refresh_token"` — it helps the server identify the token type but is optional.
- On success, the response is `{ "success": true }`.

Place this function near the other OAuth functions in the codebase.

If **No**, skip this phase entirely.

---

## Phase 7: Run Tests

After all code changes are complete:

1. **Detect the test runner** — look for `package.json` scripts, `pytest.ini`, `go test`, Makefile targets, etc.
2. **Run the existing test suite** to check for regressions
3. **Report results** — if tests fail, investigate whether the failures are related to the migration changes and fix them

If there are no existing tests, skip this step but mention it to the developer.

---

## Phase 8: Final Instructions

After all changes are made and tests pass, tell the developer:

> **Migration complete! Here's how to test:**
>
> 1. Make sure your draft version with the "New OAuth Flow" toggle is **"Active for me"**
> 2. Start your app's development server
> 3. Trigger the OAuth flow (install/re-authorize your app on the account where the draft is active)
> 4. Verify you receive both an access token and a refresh token
> 5. Make an API call with the access token to confirm it works
> 6. Wait for the token to expire (or decode the JWT to check the `exp` claim), then verify the refresh logic kicks in
> 7. Once everything works, promote the draft version to live
>
> **Key differences from the old flow to remember:**
> - Access tokens now **expire** — your app must handle refresh
> - Each refresh gives you a **new** refresh token — always store the latest one
> - The token endpoint changed from `/oauth2/token` to `/oauth_ms/oauth/token`

---

## Migration Summary Checklist

Before claiming completion, verify ALL of these:

- [ ] Old endpoint `auth.monday.com/oauth2/token` replaced with `auth.monday.com/oauth_ms/oauth/token`
- [ ] Response handling updated to extract both `access_token` and `refresh_token`
- [ ] Token storage updated to persist both tokens
- [ ] Refresh token function created and integrated
- [ ] Token expiration handling added (proactive or reactive)
- [ ] PKCE added (if chosen)
- [ ] Revocation function added (if chosen)
- [ ] Existing tests pass (if applicable)
