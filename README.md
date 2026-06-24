# Super Duper Guide

The playbook for building **Super Duper Apps** — open-source (MIT), Brale API–powered applications with a shared design language, Value Layer grammar, and CSF/GSF standards.

This repo is the collective knowledge of the family. It improves every time an app is built, debugged, or shipped.

## The family

| App | Customer focus | Runtime | Brale surface |
|-----|----------------|---------|---------------|
| [super-duper-wallet](https://github.com/superduperlabs/super-duper-wallet) | People who want a Venmo-simple, self-custody stablecoin wallet | Chrome extension (WXT) | `data.brale.xyz`, `brale.network/api` |
| [super-duper-agent-wallet](https://github.com/superduperlabs/super-duper-agent-wallet) | Developers building AI agents and bots that move stablecoins | npm library | `brale.network/api` (read fallback) |
| [super-duper-intents](https://github.com/superduperlabs/super-duper-intents) | Users and protocols moving Brale-issued stables cross-chain at par | Cloudflare Workers | `api.brale.xyz` (ION, convert, attestor) |
| [super-duper-data](https://github.com/superduperlabs/super-duper-data) | Brale customers who want a real-time analytics command center | Cloudflare Workers | `api.brale.xyz` (full orchestration) |
| [super-duper-dashboard](https://github.com/superduperlabs/super-duper-dashboard) | Brale API users who want a self-hosted neobank dashboard | Cloudflare Workers + Next.js | `api.brale.xyz` (full orchestration) |
| [super-duper-analysis](https://github.com/superduperlabs/super-duper-analysis) | Compliance teams needing transaction monitoring and investigation | Cloudflare Workers | `brale.network/api` (registry); own KYT API |

## Start here

1. [guide/getting-started.md](guide/getting-started.md) — anatomy of a Super Duper App
2. [guide/brale-api.md](guide/brale-api.md) — Brale API patterns and pitfalls
3. [guide/cloudflare-stack.md](guide/cloudflare-stack.md) — which Cloudflare primitive when
4. [guide/csf-gsf.md](guide/csf-gsf.md) — Commons Stablecoin Format and Graph Standard Format
5. [lessons/README.md](lessons/README.md) — how lessons flow back into this guide

## External references

| Reference | Location | Last verified |
|-----------|----------|---------------|
| Brale API docs index | [docs.brale.xyz/llms.txt](https://docs.brale.xyz/llms.txt) | 2026-06-24 |
| Brale Agent Kit | [superduperdot/brale-agent-kit](https://github.com/superduperdot/brale-agent-kit) | 2026-06-24 |
| CSF spec | [Brale-xyz/commons/csf.json](https://github.com/Brale-xyz/commons/blob/main/Commons%20Stablecoin%20Format/csf.json) | 2026-06-24 |
| GSF spec | [benmilne-com/standards/gsf-0.5.2.json](https://github.com/benmilne-com/standards/blob/main/gsf/gsf-0.5.2.json) | 2026-06-24 |
| Brale OpenAPI | [api.brale.xyz/openapi](https://api.brale.xyz/openapi) | 2026-06-24 |

## Templates

Copy from [templates/](templates/) when starting a new app:

- [README.md](templates/README.md) — standard README skeleton
- [brain.md](templates/brain.md) — project source of truth
- [AGENTS.md](templates/AGENTS.md) — working agreement for contributors and AI agents
- [architecture.md](templates/architecture.md) — as-built architecture
- [CONTRIBUTING.md](templates/CONTRIBUTING.md) — setup and PR conventions
- [SECURITY.md](templates/SECURITY.md) — vulnerability reporting

## Lessons by app

- [wallet.md](lessons/wallet.md) — Super Duper Wallet
- [agent-wallet.md](lessons/agent-wallet.md) — Super Duper Agent Wallet
- [intents.md](lessons/intents.md) — Super Duper Intents
- [data.md](lessons/data.md) — Super Duper Data
- [dashboard.md](lessons/dashboard.md) — Super Duper Dashboard
- [analysis.md](lessons/analysis.md) — Super Duper Analysis

## License

MIT — see [LICENSE](LICENSE).
