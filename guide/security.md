# Security

Consolidated security patterns from across the Super Duper Apps family.

## OWASP and audit

| App | Coverage |
|-----|----------|
| super-duper-data | OWASP Top 10 (2021); 47 Playwright security tests (`npm run test:security`) |
| super-duper-intents | OWASP review v1.5.1; on-chain + Worker hardening |

Run security suites before shipping public deployments.

## Secrets and encryption

| Control | Implementation | Where this is used |
|---------|----------------|-------------------|
| Credentials at rest | AES-GCM-256; `ENCRYPTION_KEY` via wrangler secret | super-duper-data, super-duper-dashboard, super-duper-analysis |
| Webhook secrets | Encrypted in D1; Base64URL-decode before HMAC | super-duper-data |
| No client-side Brale credentials | Server Actions / Workers only | super-duper-dashboard, super-duper-data |
| Secret redaction in logs | Never log client_secret or raw keys | brale-agent-kit, all apps |

Generate keys: `openssl rand -hex 32`

## Sessions and auth

| Pattern | Where this is used |
|---------|-------------------|
| D1-backed session tokens, revocable, TTL | super-duper-data (24h Bearer) |
| Server-side sessions | super-duper-dashboard |
| **Do not** use KV for auth state | `[learned: super-duper-analysis]` — KV is eventually consistent |

## Rate limiting

| App | Pattern |
|-----|---------|
| super-duper-data | 5 attempts / 15 min / IP in D1 |
| super-duper-analysis | RateLimitDO per API key |

## Webhook verification

HMAC-SHA256 over raw body bytes; constant-time compare; read body before JSON parse.

See [brale-api.md](brale-api.md) and [reference/webhook-verify.ts](https://github.com/superduperdot/brale-agent-kit/blob/main/reference/webhook-verify.ts).

## Security headers

| Header | Where this is used |
|--------|-------------------|
| HSTS (preload), CSP, X-Frame-Options: DENY | super-duper-data |
| `cache-control: no-store` on financial API responses | super-duper-intents |

## Crypto library allowlist

Use only:

- `@noble/*`, `@scure/*`, `hash-wasm`
- **Web Crypto API** in Cloudflare Workers

Do **not** use ethers. Import viem lazily in extension service workers.

`[learned: super-duper-wallet/AGENTS.md]`

## Self-custody (wallet family)

| Rule | Where this is used |
|------|-------------------|
| Decrypted keys never in `chrome.storage.local` | super-duper-wallet |
| Session key in `chrome.storage.session` (TRUSTED_CONTEXTS) | super-duper-wallet |
| Zero-fill key material after use | super-duper-wallet VaultManager, super-duper-agent-wallet `destroy()` |
| Preserve KDF salt on re-seal (regenerating bricks unlock) | super-duper-wallet ADR-007 |
| Privileged messages gated to extension pages only | super-duper-wallet ADR-008 |

## CSP pitfall (wallet)

Argon2id via `hash-wasm` requires `wasm-unsafe-eval` in MV3 CSP. Missing directive **silently broke wallet creation** until caught by E2E.

`[learned: super-duper-wallet/docs/decisions.md ADR-006]`

## Agent wallet policy

PolicyEngine before every send: caps, allowlists, audit log.

`[learned: super-duper-agent-wallet/policy.ts, audit.ts]`

## Log redaction

Apps that handle KYB/identity data must redact PII from structured logs before serialization — not just secrets, but any field that could contain personal information.

### Pattern

Use a regex that matches field **names** (not values) and replaces the value with `[REDACTED]`:

```typescript
const SENSITIVE_KEY = /ssn|ein|dob|tax_id|account_number|routing_number|phone_number|beneficial|controller|kyb|ubo/i;

function redact(obj: Record<string, unknown>): Record<string, unknown> {
  const result: Record<string, unknown> = {};
  for (const [key, value] of Object.entries(obj)) {
    if (SENSITIVE_KEY.test(key)) {
      result[key] = "[REDACTED]";
    } else if (typeof value === "object" && value !== null) {
      result[key] = redact(value as Record<string, unknown>);
    } else {
      result[key] = value;
    }
  }
  return result;
}
```

Apply to **every** log call, not just specific routes. This catches accidental logging of sensitive data anywhere in the stack.

### Where this is used

| App | Implementation |
|-----|----------------|
| **Gradient** | `log.ts` — all structured logging goes through redaction |

`[learned: gradient/apps/api/src/log.ts]`

## Passwordless email-code auth

No-password authentication using 6-digit email codes. Simpler than OAuth/SAML for B2B apps where the user population is small and controlled.

### Pattern

| Component | Detail |
|-----------|--------|
| Code generation | `crypto.getRandomValues()` → 6-digit zero-padded string |
| Storage | `email_codes` D1 table: `(id, email, code, expires_at, used, created_at)` |
| TTL | 10 minutes |
| Single-use | `UPDATE email_codes SET used = 1 WHERE id = ?` after verification |
| Rate limiting | KV counter per email: max 3 codes per 10-minute window |
| Comparison | Constant-time XOR loop (not `===`) to prevent timing attacks |
| Delivery | Resend API with verified domain; falls back to console logging in dev |
| Scope | Same code flow for login (existing user) and signup (new user) — the verify step determines the path |

### Where this is used

| App | Implementation |
|-----|----------------|
| **Gradient** | `email.ts` — `sendCode()` + `verifyCode()` |
| **super-duper-data** | D1-backed session tokens (24h Bearer) |

`[learned: gradient/apps/api/src/email.ts]`

## Internal routes

`/api/internal/*` gated by `INTERNAL_SECRET` or `X-Internal-Secret` header.

**Where this is used:** super-duper-data (`/api/internal/poll`, `/api/internal/catch-up`)

## Related

- [managed-accounts.md](managed-accounts.md) — KYB transit-only pattern, legal compliance
- [templates/SECURITY.md](../templates/SECURITY.md)
- [brale-api.md](brale-api.md) — webhook and OAuth patterns
- [testing.md](testing.md) — security test suites
