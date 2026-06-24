# Design style

Shared visual and UX conventions across Super Duper Apps.

## Principles

- **Vercel-like** — clean typography, generous whitespace, subtle borders
- **Dollar-first** — total USD prominent; individual tokens secondary
- **Value Layer in UI** — transfers always show ValueType + TransferType + Amount
- **Self-hosted assets** — no external CDNs for fonts or scripts where CSP requires it
- **Responsive** — design for 4K command center down to 320px phone

## Typography and fonts

**Geist** (sans + mono), self-hosted.

### Where this is used

- **super-duper-data** — self-hosted Geist; `font-src 'self'` CSP enforced

`[learned: super-duper-data/brain.md]`

## CSS framework

| Version | Apps |
|---------|------|
| Tailwind v4 | super-duper-dashboard, super-duper-analysis |
| Tailwind (standard) | super-duper-wallet (`@sdw/ui` package) |

## React and charts

| Stack | Apps |
|-------|------|
| React 19 + D3.js v7 | super-duper-data, super-duper-analysis, super-duper-dashboard |
| React + WXT (extension) | super-duper-wallet |
| React + Vite + wagmi/RainbowKit | super-duper-intents production UI (`ui-app/`) |

## Chain display metadata

Logos, explorer links, and display names for chains and fiat rails.

**Reference impl:** [brale-agent-kit/reference/chain-meta.ts](https://github.com/superduperdot/brale-agent-kit/blob/main/reference/chain-meta.ts)

**Where this is used:** super-duper-dashboard maintains local mapping (CoinGecko CDN for logos — Brale Network API lacks logo URLs). `[learned: super-duper-dashboard/brain.md]`

## UX patterns

| Pattern | Description | Where this is used |
|---------|-------------|-------------------|
| Recipient-first transfers | Pick person → method → amount (Cash App style) | super-duper-dashboard |
| Side panel default | Chrome side panel with popup toggle | super-duper-wallet v0.3.1+ |
| White-label | `brand.json` or org settings for name/logo/accent | super-duper-data, super-duper-analysis |
| Stale-while-revalidate balances | Cached balances, background refresh | super-duper-wallet balanceStore |
| Command center layout | KPI row + charts + live feed | super-duper-data |

## Vercel design guidelines

Referenced explicitly in **super-duper-intents** production UI review: [ui-app/review/REVIEW.md](https://github.com/superduperlabs/super-duper-intents/blob/main/ui-app/review/REVIEW.md)

## Responsive testing

**super-duper-data:** 70 Playwright tests × 10 viewport breakpoints (320px–4K).

Run: `npm run test` in super-duper-data.

## Related

- [csf-gsf.md](csf-gsf.md) — visualization standards
- [templates/README.md](../templates/README.md) — README structure
