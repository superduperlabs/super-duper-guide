# Lessons from Gradient

Source: [Gradient](https://github.com/bpmilne/gradient) — `AGENTS.md`, codebase (Phases 1–7)

Gradient is a card issuing and stablecoin settlement platform for financial institutions. It wraps the Brale orchestration API behind a treasury management portal with passwordless auth, invite-gated signup, multi-workspace support, and auto-sweep settlement.

**Stack:** Cloudflare Workers + D1 + KV + Queues · Hono + chanfana (OpenAPI from Zod) · Vite SPA + Workers Assets · pnpm monorepo

## KYB/KYC data is transit-only

- **What happened:** Managed account onboarding requires real identity data (EIN, SSN, DOB, beneficial owners) for mainnet KYB. Storing it in D1 would create a compliance liability.
- **What we learned:** Collect KYB data in the UI during the authenticated activation flow, forward it to Brale via `buildManagedKyb()`, and NEVER persist it in D1, NEVER log it. The function returns the payload but the caller only passes it through to the Brale API.
- **Guide:** [security.md §Log Redaction](../guide/security.md), [managed-accounts.md](../guide/managed-accounts.md)
- `[learned: gradient/apps/api/src/routes/activate-live.ts]`

## Sensitive field redaction in logs

- **What happened:** Standard structured logging would leak PII in production logs.
- **What we learned:** Use a regex-based redaction layer that matches field names containing: `ssn`, `ein`, `dob`, `tax_id`, `account_number`, `routing_number`, `phone_number`, `beneficial`, `controller`, `kyb`, `ubo`. Any matching key's value is replaced with `[REDACTED]` before serialization. Apply to every log call, not just specific routes.
- **Guide:** [security.md §Log Redaction](../guide/security.md)
- `[learned: gradient/apps/api/src/log.ts]`

## Auto-sweep inbound deposits to preferred hold

- **What happened:** Deposits arrive on multiple chains (Base, Canton, Solana) but treasurers want to see one unified balance in their preferred stablecoin.
- **What we learned:** On every `transfer.completed` webhook for an inbound deposit, check if the deposit's `(value_type, transfer_type)` matches the platform's preferred hold. If not, trigger a single-step Brale Transfer (cross-chain swap = burn + mint, no slippage for Brale-issued stablecoins). Tag sweeps with `note: "sweep:{original_transfer_id}"` for loop prevention — the webhook handler checks for the `sweep:` prefix and skips re-sweeping. Store preferred hold in a `platform_config` D1 table, managed via admin API.
- **Guide:** [managed-accounts.md §Auto-Sweep](../guide/managed-accounts.md)
- `[learned: gradient/apps/api/src/routes/webhooks.ts, gradient/apps/api/src/config.ts]`

## Transfer status state machine

- **What happened:** Brale transfer statuses (`pending`, `completed`, `failed`) don't map 1:1 to what a treasury operator needs to see. Direct deposits and sweep deposits have different user-facing lifecycles.
- **What we learned:** Define a local status machine: `pending` (outbound submitted, awaiting settlement) → `settling` (inbound received, sweep converting to preferred hold) → `settled` (transfer complete and in final form) → `failed`. Direct deposits (already in preferred hold) are recorded as `settled` immediately. Sweep deposits start as `settling` and upgrade to `settled` when the sweep's `transfer.completed` webhook fires. Frontend auto-polls every 5s while any transfer is `settling`.
- **Guide:** [managed-accounts.md §Transfer Status Machine](../guide/managed-accounts.md)
- `[learned: gradient/AGENTS.md §Transfer statuses]`

## Poller: two-job self-healing pattern

- **What happened:** Webhooks are the primary real-time path, but Brale webhook delivery is not guaranteed. A simple "retry stuck transfers" poll isn't enough — you also need to discover transfers the webhook never mentioned.
- **What we learned:** Run a Cron Trigger (every 60s) with two distinct jobs: **Recovery** — find transfers stuck in `settling` or `pending` in D1, check their status via the Brale API, and upgrade to `settled`/`failed`. O(stuck_transfers). **Discovery** — list recent Brale transfers for each active account, record any inbound deposits the webhook missed, and trigger sweeps. O(active_accounts). Together these guarantee deposits appear within 60s even if Brale never fires the webhook.
- **Guide:** [brale-api.md §Sync Architecture](../guide/brale-api.md), [managed-accounts.md §Poller](../guide/managed-accounts.md)
- `[learned: gradient/apps/api/src/poller.ts]`

## Passwordless email-code auth

- **What happened:** Financial institution users need secure authentication without password management overhead.
- **What we learned:** 6-digit codes via `crypto.getRandomValues()`, stored in `email_codes` D1 table with 10-minute TTL and single-use flag. Rate-limited via KV (3 codes per email per 10-minute window). Constant-time comparison loop (XOR each character, check aggregate). Resend API for delivery with verified domain. Works for both login (existing user) and signup (new user) — the code is tied to the email, and the verify step determines which path.
- **Guide:** [security.md §Passwordless Auth](../guide/security.md)
- `[learned: gradient/apps/api/src/email.ts]`

## Prefixed entity IDs

- **What happened:** Debugging across D1 tables, API responses, logs, and support conversations was slow because bare UUIDs/KSUIDs don't tell you what kind of entity they refer to.
- **What we learned:** Prefix every generated ID: `org_`, `usr_`, `mem_`, `prog_`, `acct_`, `wlt_`, `tr_`, `dst_`, `card_`, `iauth_`, `txn_`, `oba_`, `adm_`, `asess_`, `invc_`. Use a shared `generateId(prefix)` function. Keep Brale/Visa IDs internal — never expose them to users or in public API responses.
- **Guide:** [naming-conventions.md §Entity IDs](../guide/naming-conventions.md)
- `[learned: gradient/apps/api/src/id.ts]`

## Invite-gated signup with atomic consumption

- **What happened:** The platform needed controlled onboarding — not self-serve, but not manual either.
- **What we learned:** Admin generates readable invite codes (`GRD-XXXXX-XXXXX`) via a separate admin auth system (its own session tokens, own `/admin/*` routes). Signup requires an invite code. Validation is a two-step: frontend validates the code on "Continue" (before sending the email code), then the signup handler atomically consumes the code with `UPDATE invite_codes SET status='used', used_by_org_id=?, used_at=? WHERE id=? AND status='active'`. Single-use, no race conditions.
- **Guide:** [cloudflare-stack.md §Admin + Gated Access](../guide/cloudflare-stack.md)
- `[learned: gradient/apps/api/src/routes/admin-invites.ts, gradient/apps/api/src/routes/organizations.ts]`

## Multi-workspace architecture

- **What happened:** One person (e.g., a consultant or officer) manages treasury accounts at multiple institutions.
- **What we learned:** `memberships` table links users to organizations with a role (`owner`, `admin`, `member`). Session has an active `organization_id` that can be switched via `POST /session/switch` without re-authentication. WorkspaceMenu component (Linear-style dropdown) lists all workspaces the user belongs to. New workspaces created via `POST /workspaces` — provisions a new org, Brale account, wallet, and membership in one transaction.
- **Guide:** [cloudflare-stack.md §Multi-Tenancy](../guide/cloudflare-stack.md)
- `[learned: gradient/apps/api/src/routes/sessions.ts, gradient/apps/web/src/components/WorkspaceMenu.tsx]`

## Money amounts: decimal strings, never floats

- **What happened:** IEEE 754 floating point causes rounding errors in financial calculations.
- **What we learned:** Hard rule: amounts are decimal strings, two decimals for money. Never use float arithmetic for money. `parseFloat(amount).toFixed(2)` for display; Zod validation on the API boundary. The Brale API already returns `amount.value` as a string — keep it that way through the entire stack.
- **Guide:** [value-layer.md §Money Rules](../guide/value-layer.md)
- `[learned: gradient/AGENTS.md §Money]`

## User-facing terminology mapping

- **What happened:** Bank clients should never see Brale-internal vocabulary (ValueType, TransferType, Exchange, Leg).
- **What we learned:** Maintain an explicit mapping table in AGENTS.md. Public API uses `currency` + `chain`; the engine maps to `value_type` + `transfer_type`. Balance = amount of a ValueType on a wallet. Send/receive/wire = Transfer = flow of Legs. Convert/fund = Exchange. Users see USD, USDC, SBC — never "value type." Users see Solana, Canton — never "transfer type."
- **Guide:** [value-layer.md §End-User Terminology](../guide/value-layer.md)
- `[learned: gradient/AGENTS.md §Language]`

## Terms acceptance tracking

- **What happened:** Production deployment requires legal compliance — Terms of Service, Privacy Policy, and Brale End User Agreement acceptance.
- **What we learned:** Add a `terms_accepted_at TEXT` column to the users table. Signup form includes a checkbox linking to Terms, Privacy Policy, and the Brale EUA. The ISO timestamp is stored when the user checks the box (client-side) and passed to the API on account creation. Admin panel shows when each user agreed. Terms/Privacy pages are static HTML served from the public directory.
- **Guide:** [managed-accounts.md §Legal Compliance](../guide/managed-accounts.md)
- `[learned: gradient/apps/api/migrations/0022_terms_accepted.sql, gradient/apps/web/src/pages/Signup.tsx]`

## Chanfana: OpenAPI from Zod with zero overhead

- **What happened:** Needed API documentation and input validation without maintaining a separate OpenAPI spec.
- **What we learned:** Use Hono + chanfana. Every route is an `OpenAPIRoute` class with a `schema` property containing Zod schemas for request body, query params, and response shapes. Chanfana auto-generates `/openapi` docs. Validation is free — Zod schemas ARE the API contract. Chanfana validation errors return `{ errors: [...] }` with field paths, which the frontend parses for user-facing messages.
- **Guide:** [cloudflare-stack.md §Workers](../guide/cloudflare-stack.md)
- `[learned: gradient/apps/api/src/routes/*.ts]`

## CSS Modules as a Tailwind alternative

- **What happened:** For a treasury portal targeting financial institutions, a design system with a more restrained, Apple/Teenage Engineering aesthetic was preferred over Tailwind utility classes.
- **What we learned:** CSS Modules (`*.module.css`) with CSS custom properties as design tokens: `--ink`, `--muted`, `--signal`, `--elev-01`, `--line`, `--font-mono`, `--space-*`, `--control-height`. The `composes` keyword extends base classes (e.g., `.select` composes from `.input`). 14px minimum font size. No border-radius anywhere. Single accent color (`--signal`). This approach is lighter than Tailwind and enforces consistency through token constraints rather than utility classes.
- **Guide:** [design-style.md §CSS Modules Alternative](../guide/design-style.md)
- `[learned: gradient/apps/web/src/pages/*.module.css]`

## Vite proxy port mismatch

- **What happened:** Frontend API calls returned JSON parse errors in local dev.
- **What we learned:** Vite's proxy config must point to the actual wrangler dev port (`8788`), not the default (`8787`). When the proxy target is wrong, the response is an HTML error page, and `res.json()` throws "Unexpected non-whitespace character after JSON." Always verify the proxy target matches the wrangler output.
- **Guide:** [cloudflare-stack.md §Gotchas](../guide/cloudflare-stack.md)
- `[learned: gradient/apps/web/vite.config.ts]`
