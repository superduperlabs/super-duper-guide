# CSF and GSF

Two open formats for making value movement legible — temporal sequences (CSF) and structural graphs (GSF).

Both implement the same **Value Layer primitives**: ValueType, TransferType, Exchange.

## Specs

| Format | Spec | Maintainer |
|--------|------|------------|
| **CSF** (Commons Stablecoin Format) | [Brale-xyz/commons/csf.json](https://github.com/Brale-xyz/commons/blob/main/Commons%20Stablecoin%20Format/csf.json) | Brale (MIT) |
| **GSF** (Graph Standard Format) | [benmilne-com/standards/gsf/gsf-0.5.2.json](https://github.com/benmilne-com/standards/blob/main/gsf/gsf-0.5.2.json) | Ben Milne (MIT) |

Pinned in this guide as of **2026-06-24**. Verify upstream before major doc changes.

## CSF — funds flows (temporal)

**Answers:** How value steps from A → B, in order, through which participants.

- Output: Mermaid `sequenceDiagram`
- Leg format: `ValueType TransferType Amount` (e.g. `USDC Solana 5,000`)
- Density: Light / Medium / Heavy (detail level)
- Paste `csf.json` into LLM prompts to generate consistent diagrams

### Where CSF is used in the family

| App | Implementation |
|-----|----------------|
| **super-duper-analysis** | `src/engine/csf-renderer.ts`; `CsfDiagram` on Investigation Funds Flow tab; `GET /api/kyt/admin/subjects/:id/funds-flow` |
| **super-duper-intents** | Settlement legs (escrow → delivery → attestation → claim); `/api/capabilities` route pairs in Value Layer grammar |
| **super-duper-data** | Activity feed: `$AMOUNT SOURCE_VT SOURCE_TT → DEST_VT DEST_TT` |
| **super-duper-wallet** | Explorer URLs: `/tx/<transferType>/<valueType>/<hash>` on brale.network |

## GSF — relationship graphs (structural)

**Answers:** What entities exist and how they connect (non-temporal).

- Output: D3 force-directed graph from `{ view, nodes, links }` dataset
- Primitives: relationship, type, variables, weight
- Density: light / medium / heavy (Light is achromatic B&W, like CSF)
- Paste `gsf-0.5.2.json` into LLM prompts to generate datasets

### Where GSF applies

| App | Status |
|-----|--------|
| **super-duper-analysis** | Relationship graph with VT/TT edge labels implements GSF structural model at Light/Medium — hop depth 1–5 |
| **super-duper-dashboard** | Contact Profiles + payment methods — natural GSF graph (not yet visualized) |
| **super-duper-data** | Account → address → value type relationships (planned) |
| **super-duper-intents** | Solver/route network topology (planned) |

## CSF ↔ GSF interop

| CSF | GSF |
|-----|-----|
| participant | node |
| value leg (`->>`) | `transaction` or `transfers_via` link |
| data leg (`-->>`) | `instruction` link |
| `[EXCHANGE]` | `transfers_via` with differing endpoints + `via` |
| sequence order | **not represented** in GSF — keep CSF if order matters |

`CSF → GSF → CSF` loses step ordering. `GSF → CSF → GSF` is lossless for structure.

Source: [benmilne-com/standards README](https://github.com/benmilne-com/standards/blob/main/README.md)

## UI rule

Every transfer display in a Super Duper App should render as:

```
ValueType TransferType Amount
```

Never invent categories like "onramp", "offramp", or "swap" in user-facing copy — use Value Layer grammar instead.

`[learned: super-duper-data/brain.md, super-duper-dashboard/brain.md]`

## Related

- [value-layer.md](value-layer.md) — the three primitives in depth
- [design-style.md](design-style.md) — chart and visualization conventions
