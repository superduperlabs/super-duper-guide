# Naming conventions

Single source of truth for names across the Super Duper Apps family. Apply these **before** creating a repo, README, or first commit — names propagate everywhere and are costly to change later.

`[learned: super-duper-guide family rename, 2026-06-24]`

## The four layers

Every app has up to four name surfaces. They are related but not identical.

| Layer | Format | Example | Where it appears |
|-------|--------|---------|------------------|
| **Repo slug** | `super-duper-{noun}` kebab-case | `super-duper-wallet` | GitHub URL, clone path, wrangler project names (new apps) |
| **Display name** | `Super Duper {Noun}` title case | Super Duper Wallet | README H1, UI chrome, marketing, npm `description` |
| **Package name** | `@super-duper/{name}` kebab-case | `@super-duper/wallet` | `package.json` `name` field (new apps) |
| **Value Layer label** | `{ValueType} {TransferType} {Amount}` | `USDC Base 10.00` | All user-facing transfer copy |

Do not conflate layers. The repo is `super-duper-wallet`; the product is **Super Duper Wallet**; the legacy npm scope `@sdw/*` still works but new apps should use `@super-duper/*`.

## Repo naming

### Pattern

```
super-duper-{noun}
```

- **Prefix:** always `super-duper-` (not `superduper`, not `sd-`, not `better-`)
- **Noun:** short, singular or compound — what the app *is* (`wallet`, `data`, `analysis`, `intents`)
- **Case:** kebab-case only
- **Org:** `github.com/superduperlabs/super-duper-{noun}`

### Current family

| Repo slug | Display name |
|-----------|----------------|
| `super-duper-wallet` | Super Duper Wallet |
| `super-duper-agent-wallet` | Super Duper Agent Wallet |
| `super-duper-intents` | Super Duper Intents |
| `super-duper-data` | Super Duper Data |
| `super-duper-dashboard` | Super Duper Dashboard |
| `super-duper-analysis` | Super Duper Analysis |
| `super-duper-guide` | Super Duper Guide |

### Anti-patterns

| Don't | Do instead | Why |
|-------|------------|-----|
| `sdwallet`, `better-intents`, `superduperkyt` | `super-duper-{noun}` | Inconsistent; breaks family discoverability |
| `SuperDuperWallet` (camelCase slug) | `super-duper-wallet` | GitHub slugs are kebab-case |
| `super-duper-wallet-app` | `super-duper-wallet` | Drop redundant `-app` suffix |
| Rename npm/worker URLs casually | Document legacy names in README | Deployed URLs and package scopes are expensive to migrate |

## Display naming

### Pattern

```
Super Duper {Noun}
```

- Title case, spaces (not hyphens, not camelCase)
- README **H1** uses the display name: `# Super Duper Wallet`
- Beta warnings, changelogs, and UI chrome use the display name
- One-liner under H1 describes what it does — not the old internal codename

### Legacy display names (historical — do not use in new copy)

| Old | Current |
|-----|---------|
| SuperDuperWallet | Super Duper Wallet |
| SuperDuperKYT | Super Duper Analysis |
| SuperDuperData | Super Duper Data |
| better-intents | Super Duper Intents |
| @sdw/agent | Super Duper Agent Wallet |

Legacy npm scopes (`@sdw/core`, `@sdw/agent`) and worker URLs (`better-intents-ui.brale-net.workers.dev`) may remain for compatibility. README should note them explicitly under a **Legacy identifiers** subsection if relevant.

## npm package naming

### New apps

```json
{
  "name": "@super-duper/{short-name}",
  "description": "Super Duper {Noun} — one-line description"
}
```

- Scope: `@super-duper/` (publish under `superduperlabs` org when ready)
- Short name matches repo noun: `@super-duper/wallet`, `@super-duper/data`

### Monorepo packages

```
@super-duper/{app}-{layer}
```

Examples: `@super-duper/wallet-core`, `@super-duper/wallet-ui`, `@super-duper/wallet-extension`

Legacy `@sdw/*` packages in super-duper-wallet are grandfathered — do not rename without a migration plan.

## Value Layer UI copy

Never invent financial abstractions in user-facing text.

| Don't say | Say instead |
|-----------|-------------|
| onramp | `USD Wire → USDC Base` (show both legs) |
| offramp | `USDC Base → USD ACH Credit` |
| swap | `USDC Base → SBC Base` (or whatever the actual pair is) |
| payout | `SBC Base → USD ACH Credit` |
| chain | transfer type: `Base`, `Solana`, `Wire` |
| token | value type: `USDC`, `SBC`, `USD` |

Format: **`ValueType TransferType Amount`** or **`$AMOUNT SOURCE_VT SOURCE_TT → DEST_VT DEST_TT`**

See [value-layer.md](value-layer.md) and [csf-gsf.md](csf-gsf.md).

## File and doc naming

| File | Purpose |
|------|---------|
| `brain.md` | As-built project truth (lowercase, repo root) |
| `AGENTS.md` | Agent working agreement (uppercase, repo root) |
| `architecture.md` | As-built architecture (lowercase) |
| `docs/decisions.md` or `docs/decisions/` | Append-only ADR log |
| `docs/status.md` | Living feature matrix (optional but recommended) |

Do not rename these files. Agents across the family look for them by exact name.

## Cloudflare resource naming (new apps)

Prefer consistency with repo slug:

| Resource | Pattern | Example |
|----------|---------|---------|
| Worker name | `super-duper-{noun}` or `super-duper-{noun}-{role}` | `super-duper-data`, `super-duper-analysis-api` |
| D1 database | `super-duper-{noun}` | `super-duper-dashboard` |
| KV namespace | `{noun}-{purpose}` | `analysis-kv` |

Legacy worker names (`better-intents-ui`, `superduperkyt-api`) are deploy identifiers — document in README, don't silently rename.

## License and badges

- **License:** MIT for all Super Duper Apps
- **README badge line:** `**License:** [MIT](./LICENSE)`
- **Footer:** link to [Super Duper Guide](https://github.com/superduperlabs/super-duper-guide)

## Checklist before first commit

- [ ] GitHub repo created as `superduperlabs/super-duper-{noun}`
- [ ] README H1 is `Super Duper {Noun}` (display name)
- [ ] No legacy codenames in H1 or customer-focus section
- [ ] `package.json` name follows `@super-duper/*` (or legacy documented)
- [ ] Value Layer grammar in any transfer UI copy
- [ ] MIT LICENSE file present
- [ ] Entry added to family table in [README.md](../README.md)
- [ ] `lessons/{noun}.md` stub created in this guide repo

## Related

- [templates/SEED-PROMPT.md](../templates/SEED-PROMPT.md) — copy-paste bootstrap prompt for new apps
- [getting-started.md](getting-started.md) — new app checklist
- [value-layer.md](value-layer.md) — UI copy grammar
