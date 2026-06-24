# Cloudflare stack

Which Cloudflare primitive to use when, with examples from the Super Duper Apps family.

Not every app uses Cloudflare — see [getting-started.md](getting-started.md) for the family table.

## Decision guide

| Need | Primitive | Avoid |
|------|-----------|-------|
| HTTP API + static assets | **Workers** | Long-lived Node servers |
| Relational queries at edge | **D1** | KV for structured data |
| WebSocket hub, solver state, rate limit singleton | **Durable Objects** | DO for high-throughput fan-out |
| Scheduled polling | **DO Alarms** or **Cron Triggers** | In-memory timers (Workers are ephemeral) |
| Fast cache, idempotency dedup | **KV** | KV for auth sessions |
| Bulk / cold storage | **R2** | R2 on critical path |
| Async background work | **Queues** | Queues for request fan-out |
| Short-lived response cache | **Cache API** | — |
| Crypto in Workers | **Web Crypto API** | Node `crypto` module |

## Workers

Unified backend: API routes, webhooks, auth middleware, security headers, static asset serving.

### Where this is used

| App | Pattern |
|-----|---------|
| super-duper-data | Single Worker — API, webhooks, auth, SPA |
| super-duper-intents | Multiple Workers (writepath, solver, UI) via **service bindings** |
| super-duper-analysis | **Two-Worker topology**: API Worker vs Processor Worker |

**Lesson:** Queue consumers on the same Worker as live API caused latency spikes in prior systems. Separate API and processor Workers.

`[learned: super-duper-analysis/docs/decisions/, nx-validator postmortem cited in analysis brain.md]`

## D1

SQLite at the edge. Use for sessions, transfers, analytics, audit logs, rate limit counters.

### Where this is used

| App | Schema focus |
|-----|--------------|
| super-duper-data | transfers, daily_stats, sessions, audit_log, rate_limits |
| super-duper-dashboard | Full Brale mirror via Drizzle (local SQLite dev, D1 prod) |
| super-duper-intents | P&L ledger, order state, telemetry |
| super-duper-analysis | org-scoped OLTP + global intelligence tables |

**Patterns:**

- `INSERT OR REPLACE` / `INSERT OR IGNORE` for idempotent sync `[learned: super-duper-data/brain.md]`
- Pre-compute aggregates (`daily_stats`) for O(1) chart reads `[learned: super-duper-data/brain.md]`
- Smart Placement: `"placement": { "mode": "smart" }` — Worker near D1 `[learned: super-duper-data/wrangler.jsonc]`

## Durable Objects

Stateful singletons: one instance per key (org, order, API key).

### Where this is used

| App | DO | Role |
|-----|-----|------|
| super-duper-data | WebSocket hub | Hibernation + alarm polling |
| super-duper-intents | OrderCoordinator, SolverDO, SenderDO | Per-order lock, fill loop, nonce-safe EVM send |
| super-duper-analysis | ScreeningEngineDO, ContinuousMonitorDO, RateLimitDO | Per-org screen, global re-screen, per-key limits |

**Patterns:**

- `blockConcurrencyWhile` for init `[learned: super-duper-intents/brain.md]`
- `alarm()` self-scheduling for continuous loops `[learned: super-duper-analysis]`
- Cron + DO alarms replace long-lived `watchContractEvent` `[learned: super-duper-intents/brain.md]`

## DO Alarms

| App | Schedule |
|-----|----------|
| super-duper-data | 60s transfer poll, 5min balance poll |
| super-duper-analysis | Continuous re-screening |

Workers are ephemeral — never rely on in-memory timers alone.

## KV

Eventually consistent. Good for caches, bad for auth.

### Where this is used

- **super-duper-analysis** — VT/TT registry (1h TTL), idempotency, poll status
- **super-duper-intents** — hot config / kill switch

`[learned: super-duper-analysis]`

## R2

Cold storage — async writes only.

### Where this is used

- **super-duper-analysis** — access log Parquet, org logos, ingest snapshots
- **super-duper-intents** — gate run archives

## Queues

Async pipelines with retries.

### Where this is used

- **super-duper-analysis** — transfer-processing, access-log-drain (3 retries + D1 dead-letter)

**Anti-pattern:** Queues for fan-out or synchronous coordination — 0.9% delivery observed in a prior system. Use service bindings or await in Worker instead.

`[learned: super-duper-intents/brain.md]`

## Cache API

Edge-cached GET responses with short TTL.

**Where this is used:** super-duper-data — analytics responses 10–30s via `ctx.waitUntil(cache.put(...))`

**Gotcha:** `caches.default` needs a type cast in Workers TypeScript. `[learned: super-duper-data/brain.md]`

## Web Crypto

AES-GCM, PBKDF2, HMAC-SHA256 in Workers — no Node crypto.

| App | Notes |
|-----|-------|
| super-duper-data, super-duper-dashboard, super-duper-analysis | Standard Web Crypto |
| super-duper-intents | Ed25519 via WebCrypto; secp256k1 via `@noble` (WebCrypto lacks secp256k1) |

**Gotcha:** Cast `Uint8Array` as `BufferSource` for TS. `[learned: super-duper-data/brain.md]`

## ctx.waitUntil()

Non-blocking work after response (webhook ack, cache put, D1 writes).

**Rule:** `ctx.waitUntil` is **not** delivery — await anything that gates a critical state change (e.g. "filled").

`[learned: super-duper-intents/brain.md]`

## Next.js on Cloudflare

**super-duper-dashboard** uses `@opennextjs/cloudflare` + D1 (Drizzle) + KV for Next.js incremental cache.

## Gotchas (general)

| Gotcha | Mitigation | Source |
|--------|------------|--------|
| Vite plugin + custom `root` | Point `@cloudflare/vite-plugin` at `../wrangler.jsonc` | `[learned: super-duper-data/brain.md]` |
| Two local D1 DBs | Migrate both wrangler CLI and Vite plugin state dirs | `[learned: super-duper-data/brain.md]` |
| External font CDNs | Self-host Geist; `font-src 'self'` CSP | `[learned: super-duper-data/brain.md]` |
| Wrong deploy assets dir | Deploy from built output dir (e.g. `dist/brale_analytics/`) | `[learned: super-duper-data/brain.md]` |
| Poll don't subscribe on Workers | HTTP polling for EVM/Solana confirmation | `[learned: super-duper-intents/brain.md]` |

## Related

- [brale-api.md](brale-api.md) — webhook and sync patterns
- [security.md](security.md) — encryption and secrets in Workers
- [lessons/data.md](../lessons/data.md) — Cloudflare lessons from Super Duper Data
