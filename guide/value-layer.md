# Value Layer grammar

The shared domain model across all Super Duper Apps. Every movement of value is described with three primitives.

Essays and field guide: [benmilne.com/book/the-value-layer](https://benmilne.com/book/the-value-layer)

## The three primitives

| Primitive | Meaning | Example |
|-----------|---------|---------|
| **ValueType** | What moves | `USD`, `USDC`, `SBC` |
| **TransferType** | How it moves | `wire`, `ach_credit`, `base`, `solana` |
| **Exchange** | Converting between pairs | USD Wire → USDC Solana |

The **Issuer** is inherent in the ValueType (e.g. SBC is Brale-issued).

## Display format

```
$AMOUNT SOURCE_VT SOURCE_TT → DEST_VT DEST_TT
```

Example:

```
COMPLETE  $1,000  USD Wire → USDC Solana  2s ago
```

## Rules (family-wide)

1. **Never** use invented abstractions: "onramp", "offramp", "swap", "payout" in UI copy — describe the actual `(ValueType, TransferType)` legs.
2. Transfer types must be **specific**: "ACH Credit" not "ACH"; "Solana" not "chain".
3. Null or missing values show `—`, never blank.
4. Fiat legs have no on-chain hash; cross-chain may show burn + mint hashes.

`[learned: super-duper-data/brain.md]`

## In the Brale API

The API encodes the same model:

- `value_type` + `transfer_type` on source and destination
- `address_id` as the universal endpoint identifier
- `amount.currency` is always `"USD"` (unit of account), not the token being moved

See [brale-api.md](brale-api.md).

## In CSF and GSF

Both formats express the same primitives in different modalities — CSF for sequences, GSF for graphs. See [csf-gsf.md](csf-gsf.md).

## Where this is used

| App | How |
|-----|-----|
| super-duper-data | Entire dashboard — KPIs, charts, feed, filters |
| super-duper-dashboard | Transfer flows, history, audit log |
| super-duper-analysis | Screening legs, graph edge labels, CSF funds flows |
| super-duper-intents | Capabilities registry, quote engine |
| super-duper-wallet | Activity feed, send review, registry |
| super-duper-agent-wallet | Audit log entries |

## Exchange detection (analysis)

When building cross-rail logic:

| Change | Pattern name |
|--------|--------------|
| ValueType changes, same TransferType | swap |
| TransferType changes, same ValueType | bridge |
| Both change | cross-rail (fiat ↔ crypto seam) |

`[learned: super-duper-analysis/docs/decisions/]`

## Related

- [csf-gsf.md](csf-gsf.md)
- [brale-api.md](brale-api.md)
