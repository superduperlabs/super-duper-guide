# Lessons

How knowledge flows back into the [Super Duper Guide](../README.md).

## Lesson format

Each entry in an app lesson file should include:

1. **What happened** — the problem or discovery
2. **What we learned** — the pattern or anti-pattern
3. **Where it applies** — which `guide/*.md` section to read
4. **Source** — link to the app's `brain.md` entry or ADR
5. **Tag** — `[learned: super-duper-<app>/<file>]`

Example:

```markdown
### sharedSecret is Base64URL-encoded

- **What happened:** Webhook signatures always failed on first deploy.
- **What we learned:** Decode `sharedSecret` from Base64URL to raw bytes before HMAC.
- **Guide:** [brale-api.md §Webhooks](../guide/brale-api.md)
- **Source:** super-duper-data/brain.md
- `[learned: super-duper-data/brain.md]`
```

## Trigger list: when X happens, update Y

| When this happens… | …update these |
|--------------------|---------------|
| A new Brale API endpoint is used | `guide/brale-api.md` + repo README Brale API table |
| A new Cloudflare primitive is adopted | `guide/cloudflare-stack.md` + repo README |
| A bug is caused by a Brale API quirk | `guide/brale-api.md` pitfalls + `lessons/<app>.md` |
| A Cloudflare gotcha is discovered | `guide/cloudflare-stack.md` gotchas + `lessons/<app>.md` |
| A security control is added/changed | `guide/security.md` + repo README |
| CSF or GSF is implemented/extended | `guide/csf-gsf.md` |
| A new app is created | `guide/getting-started.md` + `guide/naming-conventions.md` + `templates/SEED-PROMPT.md` + new `lessons/<app>.md` |
| Brale docs change (`llms.txt`) | Verify `guide/brale-api.md` |
| `brale-agent-kit/AGENTS.md` updated | Pull new `[learned:]` into `guide/brale-api.md` |
| A `brain.md` entry is added | Export here if it helps other apps |
| An ADR is written | Add to guide if family-wide |
| `csf.json` or `gsf-*.json` updated | `guide/csf-gsf.md` pinned version |

## External references

| Reference | Location | Last verified |
|-----------|----------|---------------|
| Brale API index | [docs.brale.xyz/llms.txt](https://docs.brale.xyz/llms.txt) | 2026-06-24 |
| Brale Agent Kit | [superduperdot/brale-agent-kit](https://github.com/superduperdot/brale-agent-kit) | 2026-06-24 |
| CSF | [Brale-xyz/commons/csf.json](https://github.com/Brale-xyz/commons/blob/main/Commons%20Stablecoin%20Format/csf.json) | 2026-06-24 |
| GSF | [benmilne-com/standards/gsf-0.5.2.json](https://github.com/benmilne-com/standards/blob/main/gsf/gsf-0.5.2.json) | 2026-06-24 |

## By app

- [wallet.md](wallet.md)
- [agent-wallet.md](agent-wallet.md)
- [intents.md](intents.md)
- [data.md](data.md)
- [dashboard.md](dashboard.md)
- [analysis.md](analysis.md)
- [benchmark.md](benchmark.md)
