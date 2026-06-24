# Lessons from Super Duper Agent Wallet

Source: [super-duper-agent-wallet](https://github.com/superduperlabs/super-duper-agent-wallet) — `brain.md`, `README.md`

## Policy engine replaces human approval

- **What happened:** Agents need programmatic sends without popups.
- **What we learned:** PolicyEngine (`maxPerSend`, `maxPerSession`, allowlists) + AuditLog before every `send()`. Reject before network call.
- **Guide:** [brale-api.md §Agent policy](../guide/brale-api.md), [security.md](../guide/security.md)
- `[learned: super-duper-agent-wallet/src/policy.ts, audit.ts]`

## Brale as read fallback only

- **What happened:** Solana RPC rate limits leave stablecoin balances at zero.
- **What we learned:** `GET brale.network/api/wallet/solana/{address}` — sum in/out by `value_type` when RPC fails. Not used for sending.
- **Guide:** [brale-api.md §Read APIs](../guide/brale-api.md)
- `[learned: super-duper-agent-wallet/src/wallet.ts]`

## Independent crypto reimplementation

- **What happened:** Headless library can't depend on `@sdw/core` extension package.
- **What we learned:** Same BIP-39/44/SLIP-0010 standards as wallet; same addresses from same mnemonic; no shared npm package yet.
- **Guide:** [getting-started.md](../guide/getting-started.md)
- `[learned: super-duper-agent-wallet/brain.md]`

## Token-2022 on Solana

- **What happened:** SBC uses Token-2022 program.
- **What we learned:** Query both `TOKEN_PROGRAM_ID` and `TOKEN_2022_PROGRAM_ID` for SPL discovery.
- **Guide:** [lessons/wallet.md](wallet.md) (related RPC patterns)
- `[learned: super-duper-agent-wallet/CHANGELOG]`

## destroy() zero-fill

- **What happened:** Long-lived agent processes must release key material.
- **What we learned:** Explicit `destroy()` zero-fills; subsequent calls throw.
- **Guide:** [security.md](../guide/security.md)
- `[learned: super-duper-agent-wallet/src/wallet.ts]`
