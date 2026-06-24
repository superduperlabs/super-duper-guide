# Lessons from Super Duper Dashboard

Source: [super-duper-dashboard](https://github.com/superduperlabs/super-duper-dashboard) — `brain.md`, `README.md`

## Local-first for speed

- **What happened:** Live API calls made pages slow.
- **What we learned:** Sync Brale data into local SQLite/D1; pages read DB not API; sub-100ms loads after sync; works offline post-sync.
- **Guide:** [brale-api.md §Sync](../guide/brale-api.md)
- `[learned: super-duper-dashboard/brain.md]`

## Recipient-first transfer UX

- **What we learned:** Cash App pattern — pick person → choose payment method → amount. Contact Profiles group wallets + banks locally (not synced to Brale).
- **Guide:** [design-style.md](../guide/design-style.md)
- `[learned: super-duper-dashboard/brain.md]`

## Plaid idempotency header

- **What happened:** Plaid POSTs failed without Idempotency-Key.
- **What we learned:** All Brale POSTs including Plaid link/register require `Idempotency-Key`.
- **Guide:** [brale-api.md §Transfers](../guide/brale-api.md)
- `[learned: super-duper-dashboard CHANGELOG v0.4.0]`

## Brale JSON:API error parsing

- **What we learned:** Parse `detail` from Brale error responses for usable server action errors.
- **Guide:** [brale-api.md §Pitfalls](../guide/brale-api.md)
- `[learned: super-duper-dashboard/brain.md]`

## Drizzle dual-database

- **What we learned:** Same Drizzle schema for `better-sqlite3` (dev) and D1 (prod); migrations via drizzle-kit + wrangler.
- **Guide:** [cloudflare-stack.md §D1](../guide/cloudflare-stack.md)
- `[learned: super-duper-dashboard/wrangler.jsonc]`

## Chain logos not in Brale API

- **What we learned:** Maintain local `chain-meta.ts` with CoinGecko CDN URLs for chain logos.
- **Guide:** [design-style.md §Chain display](../guide/design-style.md)
- `[learned: super-duper-dashboard/brain.md]`

## Environment toggle at transfer-type level

- **What we learned:** Mainnet/testnet filter applies to transfer types globally, not just addresses (v0.4.0 fix).
- **Guide:** [brale-api.md §Environments](../guide/brale-api.md)
- `[learned: super-duper-dashboard/brain.md]`

## Intent-first reconciler

- **What we learned:** Write-ahead transfer intents, exponential backoff retry, background status sync — see brale-agent-kit `reconciler.ts`.
- **Guide:** [brale-api.md §Intent-first](../guide/brale-api.md)
- `[learned: super-duper-dashboard/src/lib/brale/]`

## Audit everything

- **What we learned:** Append-only audit log for every API call and user action; Value Layer formatted.
- **Guide:** [value-layer.md](../guide/value-layer.md), [security.md](../guide/security.md)
- `[learned: super-duper-dashboard/brain.md]`
