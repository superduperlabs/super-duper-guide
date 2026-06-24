# AGENTS.md

Guidance for AI agents and contributors working on **super-duper-guide** itself.

## How this guide works

This guide is the collective knowledge of the Super Duper Apps family. It improves every time an app is built, debugged, or shipped.

### Before editing

- Check [lessons/](lessons/) for context on why a pattern exists.
- Verify `[learned:]` tags point to real files — stale tags mean stale advice.
- Check the external references in [README.md](README.md) are still current.

### Adding new content

- Every non-obvious rule needs a `[learned: repo/file]` tag.
- Every pattern needs a **Where this is used** cross-reference to at least one app in the family.
- Abstract advice without a concrete example from the family is not allowed.

### Validating the guide

- Run through each **Where this is used** link — does the file still exist? Does the pattern still apply?
- Check [docs.brale.xyz/llms.txt](https://docs.brale.xyz/llms.txt) against [guide/brale-api.md](guide/brale-api.md) — any new endpoints?
- Check each app's `brain.md` for un-exported lessons.

## External references to keep current

| Reference | What to pull |
|-----------|--------------|
| [docs.brale.xyz/llms.txt](https://docs.brale.xyz/llms.txt) | Endpoint inventory, transfer types, value types, webhook events |
| [superduperdot/brale-agent-kit](https://github.com/superduperdot/brale-agent-kit) | `AGENTS.md` learned patterns, `reference/*.ts` |
| [Brale-xyz/commons/csf.json](https://github.com/Brale-xyz/commons/blob/main/Commons%20Stablecoin%20Format/csf.json) | CSF version, participants, leg format |
| [benmilne-com/standards/gsf-0.5.2.json](https://github.com/benmilne-com/standards/blob/main/gsf/gsf-0.5.2.json) | GSF version, primitives, density levels |
| [api.brale.xyz/openapi](https://api.brale.xyz/openapi) | Canonical paths, enums, response shapes |

## Trigger list: when X happens, update Y

| When this happens… | …update these |
|--------------------|---------------|
| A new Brale API endpoint is used | `guide/brale-api.md` + repo README Brale API table |
| A new Cloudflare primitive is adopted | `guide/cloudflare-stack.md` + repo README Cloudflare table |
| A bug is caused by a Brale API quirk | `guide/brale-api.md` pitfalls + `lessons/<app>.md` + `[learned:]` tag |
| A Cloudflare gotcha is discovered | `guide/cloudflare-stack.md` gotchas + `lessons/<app>.md` |
| A security control is added/changed | `guide/security.md` + repo README |
| CSF or GSF is implemented/extended | `guide/csf-gsf.md` Where this is used |
| A new app is created | `guide/getting-started.md` family table + new `lessons/<app>.md` + templates in repo |
| Brale docs change | `guide/brale-api.md` — verify endpoint references |
| `brale-agent-kit/AGENTS.md` is updated | Pull new `[learned:]` entries into `guide/brale-api.md` |
| A `brain.md` entry is added in any app | Export to `lessons/<app>.md` if it helps other apps |
| An ADR is written | Add to relevant guide page if it applies family-wide |
| `csf.json` or `gsf-*.json` updated | `guide/csf-gsf.md` pinned version + examples |

## Conventions

- Don't commit unless explicitly asked (when editing app repos on behalf of a user).
- Use `[learned: super-duper-data/brain.md]` style tags for provenance.
- Link to [brale-agent-kit/reference/](https://github.com/superduperdot/brale-agent-kit/tree/main/reference) for production TypeScript — do not duplicate code here.
