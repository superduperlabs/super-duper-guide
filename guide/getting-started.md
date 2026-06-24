# Getting started

Anatomy of a **Super Duper App** — what every app in the family shares and how to start a new one.

## Every Super Duper App

- Is **MIT licensed** and open source
- Uses the **Brale API** (`api.brale.xyz` and/or `data.brale.xyz` / `brale.network/api`)
- Speaks **Value Layer grammar**: ValueType (what moves), TransferType (how it moves), Exchange (converting between)
- References **CSF** for temporal funds flows and **GSF** for structural relationship graphs where applicable
- Ships with **`brain.md`** (project truth), **`AGENTS.md`** (working agreement), **`architecture.md`** (as-built)
- Uses **TypeScript strict**, **pnpm**, **Vitest** for unit tests, **Playwright** for E2E where UI exists
- Follows a **Vercel-like design style**: Geist fonts, Tailwind, clean dark/light, dollar-first UI

## Family cross-reference

| App | Deploy target | `api.brale.xyz` | Read API | CSF | GSF |
|-----|---------------|-----------------|----------|-----|-----|
| super-duper-wallet | Chrome extension | — | prices, wallet history | labels + explorer URLs | — |
| super-duper-agent-wallet | npm library | — | wallet history (fallback) | audit log format | — |
| super-duper-intents | Cloudflare Workers | ION, convert, attestor | — | settlement legs | planned |
| super-duper-data | Cloudflare Workers | full orchestration | — | feed labels | planned |
| super-duper-dashboard | Cloudflare + Next.js | full orchestration | chain meta | transfer display | planned |
| super-duper-analysis | Cloudflare Workers | — | VT/TT registry | native renderer | relationship graph |

## Before you write code

1. Read [naming-conventions.md](naming-conventions.md) — decide repo slug, display name, and npm scope **first**.
2. Copy [templates/SEED-PROMPT.md](../templates/SEED-PROMPT.md) into your agent session when bootstrapping.
3. Read [brale-api.md](brale-api.md) and the [Brale Agent Kit AGENTS.md](https://github.com/superduperdot/brale-agent-kit/blob/main/AGENTS.md).
4. Copy templates from [../templates/](../templates/).
5. Skim the relevant [lessons/](../lessons/) file for the app closest to yours.
6. If deploying to Cloudflare, read [cloudflare-stack.md](cloudflare-stack.md).

## New app checklist

- [ ] Names decided per [naming-conventions.md](naming-conventions.md) (repo slug, display name, npm package)
- [ ] README follows [templates/README.md](../templates/README.md) skeleton (H1 = display name)
- [ ] `brain.md`, `AGENTS.md`, `architecture.md` present
- [ ] Customer focus stated in first screen of README
- [ ] Brale API table in README (even if "none — client-side only")
- [ ] Cloudflare table in README (or explicit "none")
- [ ] CSF/GSF section in README
- [ ] Link back to this guide in README footer
- [ ] Entry added to family table in [README.md](../README.md)
- [ ] New `lessons/<app>.md` seeded from first `brain.md` entries

## Related

- [naming-conventions.md](naming-conventions.md) — repo, display, package, UI copy
- [../templates/SEED-PROMPT.md](../templates/SEED-PROMPT.md) — bootstrap prompt
- [value-layer.md](value-layer.md) — grammar in UI and code
- [design-style.md](design-style.md) — visual conventions
- [ai-agents.md](ai-agents.md) — brain.md and AGENTS.md patterns
