# Managed accounts

End-to-end patterns for apps that onboard businesses through Brale's managed account API — from signup to live settlement.

This page covers the full lifecycle that simpler integrations (read API, single-transfer) don't encounter. If your app only reads data or sends individual transfers, see [brale-api.md](brale-api.md) instead.

## Onboarding lifecycle

```
Signup → Testnet Activation → [optional: Mainnet KYB] → Live Operations
```

| Phase | What happens | Brale API surface | Data sensitivity |
|-------|-------------|-------------------|------------------|
| **Signup** | Create local org, user, wallet records; open a Brale managed account with sandbox credentials | `POST /accounts` (test keys) | No KYB data collected |
| **Testnet activation** | Submit basic business profile so the sandbox account can be activated | `POST /accounts/{id}/kyb` (sandbox values) | Safe — sandbox data only |
| **Mainnet KYB** | Collect real identity data (EIN, SSN, DOB, beneficial owners, controller) and forward to Brale | `POST /accounts/{id}/kyb` (live keys) | **Transit-only — never persist, never log** |
| **Document upload** | When Brale requests supplemental verification documents | `POST /accounts/{id}/documents` | **Transit-only** |
| **Live operations** | Transfers, deposits, sweeps, settlement on real networks | Full orchestration API | Normal operational data |

### Where this is used

| App | Implementation |
|-----|----------------|
| **Gradient** | `POST /account/activate` (testnet), `POST /account/activate/live` (mainnet), `POST /account/documents` |

`[learned: gradient/apps/api/src/routes/activate.ts, gradient/apps/api/src/routes/activate-live.ts]`

## KYB data: transit-only pattern

**Hard rule:** Real KYB data (EIN, SSN, DOB, beneficial-owner information) is collected in the UI during the authenticated activation flow, forwarded to Brale, and **never persisted in D1, never logged**.

### Implementation

1. **Build the payload in memory** — a function like `buildManagedKyb(formData)` returns the Brale API request body but never writes it to any store.
2. **Forward immediately** — the route handler calls the Brale API with the payload and returns only the result (status, KYB status).
3. **Redact in logs** — see [security.md §Log Redaction](security.md) for the field-name regex pattern.
4. **No replay** — if the request fails, the user must re-submit. There is no stored draft.

### Compensating controls

- Log redaction regex covers: `ssn`, `ein`, `dob`, `tax_id`, `account_number`, `routing_number`, `phone_number`, `beneficial`, `controller`, `kyb`, `ubo`
- The signup form (`POST /organizations`) collects NO KYB data
- The testnet activation uses sandbox values only
- The mainnet activation route requires authentication and an active testnet account

`[learned: gradient/apps/api/src/routes/activate-live.ts, gradient/apps/api/src/log.ts]`

## Auto-sweep

When deposits arrive on any supported chain, automatically convert them to the platform's preferred hold `(value_type, transfer_type)`.

### Why

Treasurers want one balance number, not a portfolio of stablecoins across chains. Auto-sweep abstracts multi-chain complexity: the bank deposits however they want, and the balance always shows in their preferred denomination.

### Pattern

```
Webhook: transfer.completed (inbound)
  → Is this already in the preferred hold?
    → Yes: record as "settled", done
    → No:  record as "settling", trigger sweep
  → Is this a sweep itself (note starts with "sweep:")?
    → Yes: upgrade original transfer to "settled", done

Sweep = single-step Brale Transfer
  source: deposit's (value_type, transfer_type)
  destination: platform's preferred (value_type, transfer_type)
  note: "sweep:{original_transfer_id}"
```

### Configuration

Store preferred hold in a `platform_config` D1 table:

| key | value | description |
|-----|-------|-------------|
| `sweep.value_type` | `SBC` | Target stablecoin |
| `sweep.transfer_type` | `canton` | Target network |

Managed via admin API (`PATCH /admin/config`, owner role required).

### Loop prevention

Tag every sweep transfer with `note: "sweep:{id}"`. The webhook handler checks for this prefix and skips re-sweeping sweep transfers.

### Wire/ACH bypass

Wire and ACH deposits skip the sweep — configure Brale Automation destinations to point directly at the preferred hold.

`[learned: gradient/apps/api/src/routes/webhooks.ts, gradient/apps/api/src/config.ts]`

## Transfer status machine

A user-facing status model that maps Brale's internal states to what a treasury operator needs to see.

| D1 status | User sees | Indicator | When |
|-----------|-----------|-----------|------|
| `pending` | pending | gray blink | Outbound send submitted, awaiting settlement |
| `settling` | settling | amber pulse | Inbound deposit received, sweep converting to preferred hold |
| `settled` | settled | green solid | Transfer complete and in final form |
| `failed` | failed | red solid | Transfer failed |

### Transition rules

- Direct deposits (already in preferred hold) → `settled` immediately
- Sweep deposits → `settling` on receipt, `settled` when sweep's `transfer.completed` fires
- Outbound sends → `pending` on submission, `settled` on `transfer.completed`
- Any `transfer.failed` → `failed`

### Frontend behavior

- Auto-poll every 5s while any transfer has status `settling`
- Global settling indicator (pulsing amber dot) in the shell header and on the balance card
- Stop polling when all transfers reach a terminal state

`[learned: gradient/AGENTS.md §Transfer statuses]`

## Poller (Cron fallback)

Webhooks are the primary real-time path. The poller is a self-healing safety net that runs every 60s via Cron Trigger.

### Two jobs

| Job | What it does | Complexity |
|-----|-------------|------------|
| **Recovery** | Find transfers stuck in `settling` or `pending`, check Brale API, upgrade to `settled`/`failed` | O(stuck_transfers) |
| **Discovery** | List recent Brale transfers for each active account, record any inbound deposits the webhook missed, trigger sweeps | O(active_accounts) |

Together, these guarantee deposits appear within 60s even if Brale never fires the webhook.

### Where this is used

| App | Pattern |
|-----|---------|
| **Gradient** | `poller.ts` — Cron Trigger every 60s, two-job architecture |
| **super-duper-data** | DO alarm 60s — similar recovery pattern |
| **super-duper-dashboard** | 60s safety-net poll |

`[learned: gradient/apps/api/src/poller.ts]`

## Legal compliance

Any production deployment that onboards businesses through Brale needs:

1. **Terms of Service** — covering the platform's role as an API wrapper, digital asset risks, FDIC/SIPC disclaimers
2. **Privacy Policy** — data collection disclosures, Brale named as a data recipient, transit-only KYB handling
3. **Brale End User Agreement** — linked at signup per [Brale's API terms](https://brale.xyz/legal/end-user-agreement)
4. **Terms acceptance tracking** — `terms_accepted_at` column on the users table, ISO timestamp set when the user checks the checkbox at signup, visible in admin panel

### Signup flow

Checkbox above the "Continue" button: "I agree to the Terms of Service, Privacy Policy, and the Brale End User Agreement" — with links to each. Button disabled until checked. Timestamp stored client-side when checked, passed to the API on account creation.

`[learned: gradient/apps/web/src/pages/Signup.tsx, gradient/apps/api/migrations/0022_terms_accepted.sql]`

## Related

- [brale-api.md](brale-api.md) — transfer and webhook patterns
- [security.md](security.md) — transit-only data, log redaction, passwordless auth
- [cloudflare-stack.md](cloudflare-stack.md) — D1, KV, Cron Triggers
- [value-layer.md](value-layer.md) — money handling rules
