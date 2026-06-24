# Seed prompt — bootstrap a new Super Duper App

Copy everything below the line into your agent session when starting a new app in the `superduperlabs` org.

---

You are building a new **Super Duper App** — an open-source (MIT), Brale API–powered application in the [superduperlabs](https://github.com/superduperlabs) family.

## Read first (in order)

1. [Super Duper Guide — getting started](https://github.com/superduperlabs/super-duper-guide/blob/main/guide/getting-started.md)
2. [Naming conventions](https://github.com/superduperlabs/super-duper-guide/blob/main/guide/naming-conventions.md) — **apply before creating anything**
3. [Brale Agent Kit AGENTS.md](https://github.com/superduperdot/brale-agent-kit/blob/main/AGENTS.md) — if the app calls `api.brale.xyz`
4. The [lessons/](https://github.com/superduperlabs/super-duper-guide/tree/main/lessons) file closest to this app type

## Naming (decide these first)

Fill in before writing code:

| Field | Your value | Rule |
|-------|------------|------|
| **Repo slug** | `super-duper-____________` | kebab-case, `super-duper-{noun}` |
| **Display name** | Super Duper ____________ | title case, spaces |
| **npm package** | `@super-duper/____________` | matches repo noun |
| **Customer** | One sentence: who is this for? | plain language, no jargon |
| **Brale surface** | `api.brale.xyz` / read API / none | see brale-api.md |
| **Cloudflare** | Workers / extension / npm only | see cloudflare-stack.md |

**UI copy rule:** never say onramp, offramp, swap, or payout — always use Value Layer grammar: `ValueType TransferType Amount`.

## Scaffold from templates

Copy from [super-duper-guide/templates/](https://github.com/superduperlabs/super-duper-guide/tree/main/templates):

- `README.md` — fill customer focus, Brale API table, Cloudflare table, CSF/GSF section
- `brain.md` — as-built truth from day one
- `AGENTS.md` — include recursive learning section (link back to guide)
- `architecture.md` — update as you build
- `CONTRIBUTING.md`, `SECURITY.md`
- `LICENSE` — MIT

## Non-negotiables

- TypeScript strict, pnpm, MIT license
- `brain.md` + `AGENTS.md` + `architecture.md` in repo root
- README sections in order: title → who this is for → what it does → Brale API → Cloudflare → CSF/GSF → quick start → docs → Part of Super Duper Apps footer
- Link to [Super Duper Guide](https://github.com/superduperlabs/super-duper-guide) in README and AGENTS.md
- When you learn something reusable → export to `super-duper-guide/lessons/{noun}.md` with `[learned: repo/file]` tag

## Stack defaults (adjust per app type)

| App type | Stack |
|----------|-------|
| Browser extension | WXT + React + Tailwind, `@noble/*` crypto, Playwright E2E |
| Cloudflare full-stack | Workers + D1 + DO, Web Crypto, Vitest + Playwright |
| Next.js on CF | `@opennextjs/cloudflare` + D1 + Drizzle |
| npm library | TypeScript strict, Vitest, no browser APIs in core |
| On-chain | Foundry + TypeScript orchestration layer |

## When done bootstrapping

- [ ] Add row to family table in [super-duper-guide/README.md](https://github.com/superduperlabs/super-duper-guide/blob/main/README.md)
- [ ] Create `super-duper-guide/lessons/{noun}.md` with initial patterns
- [ ] Add **Lessons exported** table at bottom of `brain.md` (can start empty)

## Reference apps by type

| If you're building… | Start from lessons for… |
|---------------------|-------------------------|
| Self-custody wallet / signing | [wallet.md](https://github.com/superduperlabs/super-duper-guide/blob/main/lessons/wallet.md) |
| Agent / bot programmatic wallet | [agent-wallet.md](https://github.com/superduperlabs/super-duper-guide/blob/main/lessons/agent-wallet.md) |
| Cross-chain / solver / intents | [intents.md](https://github.com/superduperlabs/super-duper-guide/blob/main/lessons/intents.md) |
| Analytics / command center | [data.md](https://github.com/superduperlabs/super-duper-guide/blob/main/lessons/data.md) |
| Neobank / ops dashboard | [dashboard.md](https://github.com/superduperlabs/super-duper-guide/blob/main/lessons/dashboard.md) |
| Compliance / monitoring / KYT | [analysis.md](https://github.com/superduperlabs/super-duper-guide/blob/main/lessons/analysis.md) |

---

*This prompt is maintained in [super-duper-guide/templates/SEED-PROMPT.md](https://github.com/superduperlabs/super-duper-guide/blob/main/templates/SEED-PROMPT.md). Update it when family conventions change.*
