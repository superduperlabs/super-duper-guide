# Lessons from Super Duper Data

Source: [super-duper-data](https://github.com/superduperlabs/super-duper-data) — `brain.md`, `README.md`

## Value Layer grammar — no invented categories

- **What we learned:** Never "onramp/offramp/swap" in UI. Always `ValueType TransferType Amount`. Filters apply to either leg.
- **Guide:** [value-layer.md](../guide/value-layer.md)
- `[learned: super-duper-data/brain.md]`

## sharedSecret Base64URL decode

- **What happened:** Webhook HMAC verification failed until secret was decoded correctly.
- **What we learned:** `sharedSecret` from subscription is Base64URL; decode to raw bytes for HMAC key. Shown once at create time.
- **Guide:** [brale-api.md §Webhooks](../guide/brale-api.md)
- `[learned: super-duper-data/brain.md]`

## GET /accounts shape variance

- **What happened:** API returned full objects or bare KSUID strings.
- **What we learned:** Defensive parsing for all response envelope shapes.
- **Guide:** [brale-api.md §Pitfalls](../guide/brale-api.md)
- `[learned: super-duper-data/brain.md]`

## Self-healing sync (webhooks + poll + catch-up)

- **What we learned:** Webhooks primary; DO alarm 60s poll; cold-start catch-up from `MAX(created_at)`; return 200 then `ctx.waitUntil()` for D1 writes.
- **Guide:** [brale-api.md §Sync](../guide/brale-api.md), [cloudflare-stack.md](../guide/cloudflare-stack.md)
- `[learned: super-duper-data/brain.md]`

## daily_stats pre-aggregation

- **What we learned:** Pre-compute daily aggregates for O(1) chart reads vs scanning all transfers.
- **Guide:** [cloudflare-stack.md §D1](../guide/cloudflare-stack.md)
- `[learned: super-duper-data/brain.md]`

## Cloudflare deploy gotchas

- **What we learned:** Vite plugin root → point at `../wrangler.jsonc`; two local D1 DBs to migrate; deploy from `dist/brale_analytics/`; self-host fonts for CSP.
- **Guide:** [cloudflare-stack.md §Gotchas](../guide/cloudflare-stack.md)
- `[learned: super-duper-data/brain.md]`

## OWASP + 47 security tests

- **What we learned:** Full security test suite for auth, CSRF, rate limits, WebSocket auth, headers, webhook integrity.
- **Guide:** [security.md](../guide/security.md), [testing.md](../guide/testing.md)
- `[learned: super-duper-data/README.md]`

## On-chain hash display

- **What we learned:** Transfers may have 0, 1, or 2 hashes in `raw_json`; cross-chain shows burn + mint; fiat legs none.
- **Guide:** [value-layer.md](../guide/value-layer.md)
- `[learned: super-duper-data/brain.md]`

## brain.md as canonical lessons doc

- **What we learned:** When no `docs/` directory exists, brain.md holds all operational knowledge — keep it current.
- **Guide:** [ai-agents.md](../guide/ai-agents.md)
- `[learned: super-duper-data/brain.md]`
