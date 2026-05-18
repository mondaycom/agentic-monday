# monday-oauth-migrate

Migrate monday.com apps from the old OAuth flow to the new OAuth 2.1 flow with refresh tokens and optional PKCE.

## What This Plugin Does

This skill guides you through migrating an existing monday.com OAuth integration to the new OAuth 2.1 flow. The migration involves:

- Changing the token endpoint from `/oauth2/token` to `/oauth_ms/oauth/token`
- Adding refresh token support (access tokens now expire)
- Updating token storage to persist both access and refresh tokens
- Optionally adding PKCE (Proof Key for Code Exchange) for enhanced security
- Optionally adding token revocation support

## When to Use

Use this skill when:
- You need to migrate from the old monday OAuth to OAuth 2.1
- Your code calls `auth.monday.com/oauth2/token`
- You're seeing OAuth deprecation warnings
- You need to add refresh token support to your monday app

Do NOT use for:
- Implementing OAuth from scratch (use monday SDK docs instead)
- Non-monday OAuth providers (Google, Slack, etc.)
- Client credentials flow
- General monday.com debugging

## Usage

Simply tell Claude Code to migrate your OAuth flow:

```
Migrate my monday OAuth to the new flow
```

Or the skill will auto-trigger when it detects relevant patterns like:
- "update monday token endpoint"
- "add refresh tokens to my monday app"
- "oauth2/token deprecation"

## Migration Phases

1. **Detection** — Scans codebase for existing OAuth usage
2. **Dev Center Setup** — Guides you through enabling the new flow in monday Developer Center
3. **PKCE Decision** — Optional security enhancement
4. **Token Exchange Migration** — Updates the token endpoint
5. **Token Storage** — Ensures refresh tokens are persisted
6. **Refresh Logic** — Adds token refresh handling
7. **Revocation** — Optional token cleanup support
8. **Testing** — Validates the migration

## Requirements

- An existing monday.com app using OAuth
- Access to the monday.com Developer Center
- Ability to create a draft version of your app
