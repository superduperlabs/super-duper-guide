# {{App Name}}

One-line description of what this app is.

**Version:** x.y.z · **License:** [MIT](./LICENSE)

## Who this is for

Plain statement of the core customer and the problem this app solves.

## What it does

- Feature one
- Feature two
- Feature three

## Brale API usage

| Surface | Endpoint / feature | Purpose |
|---------|---------------------|---------|
| `api.brale.xyz` | `POST /accounts/{id}/transfers` | Example |
| `data.brale.xyz` | `/price/list` | Example |
| `brale.network/api` | `/wallet/...` | Example |

Or: **None** — this app does not call the Brale orchestration API (explain what it uses instead).

## Cloudflare stack

| Primitive | Usage |
|-----------|-------|
| Workers | … |
| D1 | … |

Or: **None** — browser extension / npm library / other runtime.

## Standards: CSF and GSF

How this app uses, produces, or displays CSF funds flows and GSF relationship graphs.

- [CSF spec](https://github.com/Brale-xyz/commons/blob/main/Commons%20Stablecoin%20Format/csf.json)
- [GSF spec](https://github.com/benmilne-com/standards/blob/main/gsf/gsf-0.5.2.json)

## Quick start

```bash
pnpm install
pnpm dev
```

## Architecture

Brief overview or link to [architecture.md](./architecture.md).

## Documentation

- [brain.md](./brain.md) — project source of truth (**read first**)
- [AGENTS.md](./AGENTS.md) — working agreement for contributors and AI agents
- [architecture.md](./architecture.md) — as-built architecture
- [CONTRIBUTING.md](./CONTRIBUTING.md)
- [SECURITY.md](./SECURITY.md)

## Part of Super Duper Apps

This app is part of the [Super Duper Apps](https://github.com/superduperlabs/super-duper-guide) family — open-source, MIT-licensed applications built on the [Brale API](https://docs.brale.xyz) with shared design conventions and Value Layer grammar.

See the [Super Duper Guide](https://github.com/superduperlabs/super-duper-guide) for patterns, pitfalls, and lessons learned across the family.
