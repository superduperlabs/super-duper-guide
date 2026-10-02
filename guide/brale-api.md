# Brale API integration

Patterns for the authenticated core API (`api.brale.xyz`), the read API (`brale.network/api`, `data.brale.xyz`), and production pitfalls.

**Canonical procedural docs:** [Brale Agent Kit AGENTS.md](https://github.com/superduperdot/brale-agent-kit/blob/main/AGENTS.md)  
**Reference TypeScript:** [brale-agent-kit/reference/](https://github.com/superduperdot/brale-agent-kit/tree/main/reference)

## Mental model

Everything is a **transfer** between `(value_type × transfer_type)` endpoints addressed by `address_id`. There is no separate `/swap`, `/mint`, or `/offramp` endpoint — the operation is implied by source and destination pairs.

`[learned: brale-agent-kit/AGENTS.md §3]`

## Authentication

OAuth2 client credentials → `POST https://auth.brale.xyz/oauth2/token` → Bearer token on all `api.brale.xyz` calls.

| Pattern | When to use | Reference impl |
|---------|-------------|----------------|
| Cache token; refresh ~5 min before expiry | All server integrations | [reference/auth.ts](https://github.com/superduperdot/brale-agent-kit/blob/main/reference/auth.ts) |
| De-dupe concurrent token refreshes | High-traffic Workers | `[learned: super-duper-intents/src/brale/auth.ts]` |
| Invalidate cache and retry once on 401 | Token expiry edge cases | `[learned: brale-agent-kit/reference/client.ts]` |

### Where this is used

- **super-duper-data** — Worker OAuth in setup wizard and sync
- **super-duper-dashboard** — Server Actions, token cached server-side
- **super-duper-intents** — `src/brale/auth.ts`

## Transfers and idempotency

`POST /accounts/{account_id}/transfers` with **`Idempotency-Key` header required** on every POST that creates a resource.

| Pattern | Where this is used |
|---------|-------------------|
| Fresh UUID per new operation | super-duper-data (webhook subscribe), super-duper-dashboard (Plaid, transfers) |
| Deterministic key (e.g. orderId) for safe retries | super-duper-intents |
| Same key when retrying after timeout/5xx | super-duper-dashboard reconciler pattern — see [reference/reconciler.ts](https://github.com/superduperdot/brale-agent-kit/blob/main/reference/reconciler.ts) |

**Hard rules:**

- `source` must be an **internal (custodial)** address; external addresses are destination-only. `[learned: super-duper-intents/scripts/brale-source-dryrun.mjs]`
- `amount.currency` is always `"USD"`; `amount.value` is a string. `[learned: brale-agent-kit/AGENTS.md]`
- Balance query requires both params: `?transfer_type=base&value_type=SBC`. `[learned: super-duper-intents]`

## Webhooks

Subscribe via `POST /accounts/{id}/webhooks`. Verify with HMAC-SHA256 over **raw body bytes**.

| Step | Detail |
|------|--------|
| Header | `x-request-signature-sha-256` (lowercase hex) |
| Key | **Base64URL-decode** the `sharedSecret` from subscription — do not HMAC the encoded string |
| Compare | Constant-time equality |
| Handler | Return 200 immediately; process async with `ctx.waitUntil()` |

`[learned: super-duper-data/brain.md, brale-agent-kit/AGENTS.md §6]`

### Where this is used

- **super-duper-data** — `src/worker.ts`; sharedSecret decode pitfall documented in brain.md
- **super-duper-dashboard** — webhook handler + 60s safety-net poll
- **super-duper-intents** — HMAC webhook → EIP-712 re-sign for on-chain attestation (contracts cannot verify HMAC)

**Reference impl:** [reference/webhook-verify.ts](https://github.com/superduperdot/brale-agent-kit/blob/main/reference/webhook-verify.ts) (Node + Web Crypto)

## Sync architecture

Three layers for production integrations:

| Layer | When | Where this is used |
|-------|------|-------------------|
| Full backfill | Cold start | super-duper-data, super-duper-dashboard |
| Webhooks | Real-time | super-duper-data, super-duper-dashboard |
| Safety-net poll (30–60s) | Missed events | super-duper-data (DO alarm 60s), super-duper-dashboard |

`[learned: super-duper-dashboard/src/lib/db/sync.ts, super-duper-data/brain.md §Sync]`

## Read APIs (unauthenticated)

| Endpoint | Purpose | Where this is used |
|----------|---------|-------------------|
| `data.brale.xyz/price/list` | USD price oracle | super-duper-wallet `prices.ts` |
| `brale.network/api/wallet/{chain}/{address}` | Decoded stablecoin history | super-duper-wallet `activity.ts`, super-duper-agent-wallet balance fallback |
| `brale.network/api` | Chain metadata, VT/TT registry | super-duper-dashboard `chain-meta.ts`, super-duper-analysis KV cache |

## Balance discovery

The API does not list valid `(value_type × transfer_type)` pairs per address. Probe combinations and skip errors.

**Reference impl:** [reference/balance-discovery.ts](https://github.com/superduperdot/brale-agent-kit/blob/main/reference/balance-discovery.ts)  
**Where this is used:** super-duper-dashboard `fetchAllBalances()`

## Intent-first transfers (write-ahead)

Write transfer intent to local store before calling API; reconcile with exponential backoff.

**Reference impl:** [reference/reconciler.ts](https://github.com/superduperdot/brale-agent-kit/blob/main/reference/reconciler.ts)  
**Where this is used:** super-duper-dashboard

## Pitfalls

| Pitfall | Fix | Source |
|---------|-----|--------|
| `GET /accounts` returns objects or bare KSUID strings | Defensive parsing | `[learned: super-duper-data/brain.md]` |
| Transfer IDs are UUIDs; other IDs are KSUIDs | Don't assume one format | `[learned: super-duper-data/brain.md]` |
| `sharedSecret` Base64URL-encoded, shown once | Decode before HMAC | `[learned: super-duper-data/brain.md]` |
| Status `complete` vs `completed` | Match both | `[learned: super-duper-intents, brale-agent-kit]` |
| Value types case-sensitive (`cfUSD` not `CFUSD`) | Registry-driven, never hardcode | `[learned: super-duper-intents]` |
| Brale JSON:API errors need explicit parsing | Surface `detail` in UI | `[learned: super-duper-dashboard/brain.md]` |
| Webhook OAuth scopes | Need `webhooks:read` + `webhooks:write` | `[learned: super-duper-data/brain.md]` |

## Environments

Testnet vs mainnet is determined by **which API client** minted the token, not URL path.

**Reference impl:** [reference/networks.ts](https://github.com/superduperdot/brale-agent-kit/blob/main/reference/networks.ts)

### Where this is used

- **super-duper-intents** — `REGISTRY_ENV` loads `coins.testnet.yaml` vs mainnet
- **super-duper-dashboard** — global mainnet/testnet toggle
- **super-duper-data** — testnet transfer type detection on either leg

## Managed accounts

For apps that onboard businesses through Brale's managed account API (not just send individual transfers), see the dedicated **[managed-accounts.md](managed-accounts.md)** page, which covers:

- Onboarding lifecycle (signup → testnet → mainnet KYB → live operations)
- KYB data transit-only pattern (never persist, never log)
- Auto-sweep architecture (inbound deposits → preferred hold)
- Transfer status state machine (pending → settling → settled → failed)
- Two-job poller for self-healing webhook resilience
- Legal compliance (Terms, Privacy Policy, Brale EUA acceptance tracking)

### Where this is used

| App | Managed account scope |
|-----|----------------------|
| **Gradient** | Full lifecycle: multi-workspace B2B treasury with KYB, auto-sweep, poller |
| **super-duper-dashboard** | Transfers + safety-net poll (no KYB, no sweep) |
| **super-duper-data** | Sync + webhooks + alarm poll (no KYB, no sweep) |

`[learned: gradient/AGENTS.md, gradient/apps/api/src/routes/webhooks.ts]`

## Agent policy layer

For agents that move money: PolicyEngine + AuditLog before every transfer.

**Where this is used:** super-duper-agent-wallet `policy.ts`, `audit.ts`  
`[learned: super-duper-agent-wallet/brain.md]`

## Related

- [managed-accounts.md](managed-accounts.md) — full managed account lifecycle, auto-sweep, KYB compliance
- [cloudflare-stack.md](cloudflare-stack.md) — Workers deployment patterns
- [value-layer.md](value-layer.md) — ValueType and TransferType in UI
- [brale-agent-kit](https://github.com/superduperdot/brale-agent-kit) — full endpoint reference and reference code
