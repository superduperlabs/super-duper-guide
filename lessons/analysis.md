# Lessons from Super Duper Analysis

Source: [super-duper-analysis](https://github.com/superduperlabs/super-duper-analysis) — `brain.md`, `CHANGELOG.md`, `docs/decisions/`

## Two-Worker topology (API vs Processor)

- **What happened:** Queue consumers on API Worker caused latency spikes in prior system (nx-validator).
- **What we learned:** Separate `superduperkyt-api` and `superduperkyt-processor` Workers; Queues for async scoring only.
- **Guide:** [cloudflare-stack.md §Workers](../guide/cloudflare-stack.md)
- `[learned: super-duper-analysis/docs/decisions/ADR-001]`

## Value Layer as screening spine

- **What we learned:** Every movement is a leg `(ValueType, TransferType, Amount)`; exchanges join into FundsFlows; Chainalysis `{network, asset}` maps to `{transferType, valueType}` via brale.network registry in KV.
- **Guide:** [value-layer.md](../guide/value-layer.md), [csf-gsf.md](../guide/csf-gsf.md)
- `[learned: super-duper-analysis/brain.md]`

## Native CSF renderer

- **What we learned:** CSF v1.4.7 Light Mermaid diagrams in Investigation workspace; `src/engine/csf-renderer.ts`; `GET .../funds-flow`.
- **Guide:** [csf-gsf.md](../guide/csf-gsf.md)
- `[learned: super-duper-analysis/docs/csf.json, brain.md]`

## GSF-shaped relationship graph

- **What we learned:** Hop-depth graph with VT/TT edge labels implements GSF structural model at Light/Medium — formal GSF dataset export planned.
- **Guide:** [csf-gsf.md §GSF](../guide/csf-gsf.md)
- `[learned: super-duper-analysis/brain.md v0.2.0]`

## DO patterns from production

- **What we learned:** ScreeningEngineDO (per-org), ContinuousMonitorDO (alarm re-screen), RateLimitDO (per key); don't use DOs for throughput — use Queues.
- **Guide:** [cloudflare-stack.md §Durable Objects](../guide/cloudflare-stack.md)
- `[learned: super-duper-analysis/docs/decisions/ADR-003]`

## EVM addresses always lowercase

- **What we learned:** Sanctions matching false negative = legal liability; normalize to lowercase.
- **Guide:** [security.md](../guide/security.md)
- `[learned: super-duper-analysis/docs/decisions/ADR-002]`

## Exchange detection rules

- **What we learned:** VT change + same TT = swap; TT change + same VT = bridge; both change = cross-rail.
- **Guide:** [value-layer.md](../guide/value-layer.md)
- `[learned: super-duper-analysis/docs/decisions/ADR-002]`

## KV not for auth

- **What we learned:** KV eventually consistent — fine for VT/TT registry cache, not session/auth state.
- **Guide:** [cloudflare-stack.md §KV](../guide/cloudflare-stack.md), [security.md](../guide/security.md)
- `[learned: super-duper-analysis/docs/decisions/ADR-003]`

## OpenSanctions licensing

- **What we learned:** Commercial license required for production use of OpenSanctions data.
- **Guide:** [analysis README](https://github.com/superduperlabs/super-duper-analysis) operational notes
- `[learned: super-duper-analysis/CHANGELOG v0.2.0]`

## kyt-feed training data

- **What we learned:** Import production-shaped transfers via `tools/kyt-feed` DuckDB CLI for analyst walkthroughs.
- **Guide:** [testing.md](../guide/testing.md)
- `[learned: super-duper-analysis/README.md]`
