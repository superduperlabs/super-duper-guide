# Lessons from Super Duper Wallet

Source: [super-duper-wallet](https://github.com/superduperlabs/super-duper-wallet) — `brain.md`, `docs/decisions.md`, `AGENTS.md`

## Gasless revert (ADR-014)

- **What happened:** ERC-4337 + SBC paymaster shipped; dapps broke (Uniswap expects EIP-7702, not counterfactual accounts).
- **What we learned:** Prove EOA parity before smart-wallet differentiation. Gasless deferred to future EIP-7702 + v0.8 paymaster fork.
- **Guide:** [ai-agents.md](../guide/ai-agents.md) (ADR discipline)
- `[learned: super-duper-wallet/docs/decisions.md ADR-014]`

## CSP wasm-unsafe-eval (ADR-006)

- **What happened:** Wallet creation silently failed — Argon2id via `hash-wasm` needs `wasm-unsafe-eval` in MV3 CSP.
- **What we learned:** E2E against built extension catches CSP failures unit tests miss.
- **Guide:** [security.md](../guide/security.md), [testing.md](../guide/testing.md)
- `[learned: super-duper-wallet/docs/decisions.md ADR-006]`

## Storage tiers and zero-fill

- **What happened:** Vault crypto design for self-custody extension.
- **What we learned:** Encrypted envelope in `local`; session key in `session` (TRUSTED_CONTEXTS); decrypted material heap-only, zero-filled after use. Never regenerate KDF salt on re-seal.
- **Guide:** [security.md](../guide/security.md)
- `[learned: super-duper-wallet/architecture.md, ADR-007]`

## Multi-source balances and activity

- **What happened:** Public Solana RPCs rate-limit or omit tx data.
- **What we learned:** PublicNode for SOL; race RPC vs Brale API for SPL; merge activity on `chain:txHash`; shared Zustand balance store with stale-while-revalidate.
- **Guide:** [brale-api.md](../guide/brale-api.md) (read API), [design-style.md](../guide/design-style.md)
- `[learned: super-duper-wallet/brain.md §4–5]`

## Privileged message gating (ADR-008)

- **What happened:** Content scripts must not reach vault control plane.
- **What we learned:** `sdw:popup` only from trusted extension pages; dapp origin from `sender.origin`.
- **Guide:** [security.md](../guide/security.md)
- `[learned: super-duper-wallet/docs/decisions.md ADR-008]`

## Document layer template

- **What we learned:** brain.md + ADR log + status matrix + AGENTS.md is the best agent handoff in the family.
- **Guide:** [ai-agents.md](../guide/ai-agents.md), [templates/](../templates/)
- `[learned: super-duper-wallet/AGENTS.md, brain.md]`

## CSF-aware explorer URLs

- **What we learned:** Link to `brale.network/tx/<transferType>/<valueType>/<hash>` for tracked stablecoins.
- **Guide:** [csf-gsf.md](../guide/csf-gsf.md)
- `[learned: super-duper-wallet/packages/extension/src/entrypoints/popup/lib/explorers.ts]`
