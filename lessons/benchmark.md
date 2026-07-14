# Lessons from Super Duper Benchmark

Source: [super-duper-benchmark](https://github.com/superduperlabs/super-duper-benchmark) — `brain.md`, `README.md`, `architecture.md`

## Loop-forever runs need durable scheduling, not `setTimeout`

- **What happened:** In the original Vercel implementation, loop-forever benchmark runs were driven by a long-running `while(true)` in an unawaited async function fired from a POST handler. Any redeploy, isolate eviction, or serverless timeout silently killed the loop mid-iteration; the DB was left with a stale `pendingTransferId` and no scheduler to advance it.
- **What we learned:** Put the run state machine in a Durable Object and schedule the next tick via `ctx.storage.setAlarm()`. Loop iterations survive DO eviction because the alarm re-hydrates the DO and D1 has the truth. Never rely on in-process timers on serverless runtimes.
- **Guide:** [cloudflare-stack.md §Durable Objects](../guide/cloudflare-stack.md), [brale-api.md](../guide/brale-api.md)
- `[learned: super-duper-benchmark/brain.md]`

## Deterministic per-iteration amounts make loop-mode debuggable

- **What we learned:** Loops that re-use the same transfer amount blur together in logs. Seed a random amount in `[$1, $3]` per iteration from `(runId, iteration)` so each pass through the path is visually distinct in dashboards but still reproducible from the same seed.
- **Guide:** [ai-agents.md](../guide/ai-agents.md) (deterministic-first patterns)
- `[learned: super-duper-benchmark/src/lib/amount.ts]`

## Multi-keyset webhook secret support is a first-class need

- **What happened:** Brale's `sharedSecret` is shown once and can be rotated. Attempting a rotation without dual-accepting both old and new secrets caused a receiver blackout window.
- **What we learned:** Accept a **list** of secrets. Env `BRALE_WEBHOOK_SECRET` is comma-separated; D1 `webhook_shared_secret_a` and `webhook_shared_secret_b` are both read. Verify against every candidate; report which matched.
- **Guide:** [brale-api.md §Webhooks](../guide/brale-api.md), [security.md](../guide/security.md)
- `[learned: super-duper-benchmark/src/lib/verify.ts]`

## HMAC on the raw ArrayBuffer, always

- **What we learned:** `POST /webhooks/brale` reads `await request.arrayBuffer()` and only then `JSON.parse(new TextDecoder().decode(bytes))`. Parsing to JSON first (and then re-stringifying to sign) breaks HMAC verification on payloads with insignificant whitespace or non-canonical field ordering.
- **Guide:** [brale-api.md §Webhooks](../guide/brale-api.md), [security.md](../guide/security.md)
- `[learned: super-duper-benchmark/src/routes/webhook.ts]`

## Prisma-Neon → Drizzle-D1 migration playbook

- **What we learned:** Concrete diff for anyone else porting a Prisma+Postgres app to D1:
  - `TIMESTAMP(3)` → `INTEGER` (unix ms) — simpler math than TEXT, works in Drizzle via `mode: "timestamp_ms"`
  - `BOOLEAN` → `INTEGER` (0/1) — Drizzle `mode: "boolean"` handles the mapping transparently
  - Prisma implicit indexes → explicit `index()` in Drizzle; enable `PRAGMA foreign_keys=ON`
  - `prisma db push` workflow → `drizzle-kit generate` + `wrangler d1 migrations apply`
  - One-shot data migration: stream from `pg`, emit `INSERT OR REPLACE` SQL chunks of ≤40 rows, feed to `wrangler d1 execute --file=`
- **Guide:** [cloudflare-stack.md §D1](../guide/cloudflare-stack.md)
- `[learned: super-duper-benchmark/scripts/migrate-neon-to-d1.ts]`

## OAuth token cache belongs in KV, not in-process

- **What we learned:** Node/Vercel let us keep an in-memory `Map` of `keyset -> token`. On Workers every isolate would independently refetch, blowing up latency and API calls. Move the token cache to KV with `expirationTtl = ttl - 300s` — the refresh window ensures freshness without a request wall.
- **Guide:** [brale-api.md §Auth](../guide/brale-api.md), [cloudflare-stack.md §KV](../guide/cloudflare-stack.md)
- `[learned: super-duper-benchmark/src/brale/auth.ts]`

## Copy-full-transfer-ID as a UX pattern

- **What we learned:** Support engineers spend an outsized amount of time hunting for full transfer IDs to feed into Brale support tickets. Show the full ID inline, next to a tiny "copy" button (native `navigator.clipboard.writeText`, no library needed) — the payoff was immediate.
- **Guide:** [design-style.md](../guide/design-style.md)
- `[learned: super-duper-benchmark/src/ui/pages/run-detail.tsx]`

## Server-rendered JSX + tiny islands beats React SPA for admin tools

- **What we learned:** The Next.js app was heavy for what is fundamentally a set of tables + a couple of forms. Hono's server-rendered JSX with pinpoint inline `<script>` islands (copy button, 3-second auto-refresh on Run Detail, CSV export) delivered the same UX with zero client-side hydration and a ~80KiB gzipped Worker.
- **Guide:** [design-style.md](../guide/design-style.md), [cloudflare-stack.md §UI](../guide/cloudflare-stack.md)
- `[learned: super-duper-benchmark/src/ui/]`
