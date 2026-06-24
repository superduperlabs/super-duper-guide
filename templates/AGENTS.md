# AGENTS.md

Guidance for AI agents and contributors. Read [brain.md](./brain.md) first.

## Before you start

- Read [brain.md](./brain.md) and [architecture.md](./architecture.md).
- Read [naming-conventions.md](https://github.com/superduperlabs/super-duper-guide/blob/main/guide/naming-conventions.md) before creating repos or UI copy.
- Read the [Super Duper Guide](https://github.com/superduperlabs/super-duper-guide).
- Use [SEED-PROMPT.md](https://github.com/superduperlabs/super-duper-guide/blob/main/templates/SEED-PROMPT.md) when bootstrapping a new app in the family.
- Read [Brale Agent Kit AGENTS.md](https://github.com/superduperdot/brale-agent-kit/blob/main/AGENTS.md) if this app uses the Brale API.

## Build & verify

```bash
pnpm -r typecheck
pnpm test
pnpm build
```

A change is "done" only when the above are green (add E2E if applicable).

## Conventions

- TypeScript strict; avoid `any` without reason.
- Crypto: `@noble/*`, `@scure/*`, `hash-wasm`, Web Crypto (Workers).
- Comments explain *why*, not *what*.
- Match existing naming, types, and UI patterns.

## Guardrails

- Don't commit unless explicitly asked.
- Update brain.md when what's built changes.
- Record unfixed bugs in brain.md §5.

## Recursive learning

This app is part of the [Super Duper Apps](https://github.com/superduperlabs/super-duper-guide) family.

### When you learn something new

1. Add it to this repo's `brain.md` (dated, append-only).
2. If it helps someone building a *different* Super Duper App:
   - Add to `superduperlabs/super-duper-guide/lessons/<this-app>.md`
   - Cross-reference in the relevant `guide/*.md` file
   - Tag: `[learned: <this-repo>/<file>]`
3. If it's a Brale API pattern, consider updating [brale-agent-kit/AGENTS.md](https://github.com/superduperdot/brale-agent-kit/blob/main/AGENTS.md).

### When things break

- Brale API behavior changed → update guide `brale-api.md` + brale-agent-kit.
- Cloudflare behavior changed → update guide `cloudflare-stack.md`.
- Test caught a regression → document in `lessons/<this-app>.md`.
