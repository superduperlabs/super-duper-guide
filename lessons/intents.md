# Lessons from Super Duper Intents

Source: [super-duper-intents](https://github.com/superduperlabs/super-duper-intents) — `brain.md`, `seed.md`, `plan.md`

## Issuer-native settlement is simpler than generic oracles

- **What happened:** LiFi/Polymer oracle stack: 21 txs, 13 reverts, 0 settlements for Brale-issued tokens.
- **What we learned:** Brale is the issuer — liquidity is mint/burn at par, not AMM depth. Cross-chain = burn/mint (ION), not bridge-and-prove.
- **Guide:** [brale-api.md](../guide/brale-api.md)
- `[learned: super-duper-intents/seed.md Part 2]`

## The 10 laws (machine-enforced habits)

- **What we learned:** Mined ≠ successful; sleep ≠ verification; bytecode > docs; chain > config for decimals; events > WebSocket; safety state survives restart (D1/DO); simulate before broadcast; explicit gas on dependent txs; every fill leg from the order.
- **Guide:** [testing.md](../guide/testing.md), [cloudflare-stack.md](../guide/cloudflare-stack.md)
- `[learned: super-duper-intents/seed.md §6]`

## ctx.waitUntil is not delivery

- **What happened:** Assumed background work in waitUntil would complete before marking filled.
- **What we learned:** Await anything that gates critical state (filled, claimed).
- **Guide:** [cloudflare-stack.md](../guide/cloudflare-stack.md)
- `[learned: super-duper-intents/brain.md]`

## Queues for fan-out fail

- **What happened:** ~0.9% delivery using Queues for coordination.
- **What we learned:** Queues = write-behind only; use service bindings or await in Worker.
- **Guide:** [cloudflare-stack.md §Queues](../guide/cloudflare-stack.md)
- `[learned: super-duper-intents/brain.md, nx-validator postmortem]`

## Deposit-first, custodial-zero (Layer 3)

- **What happened:** Brale can only source from internal custodial wallets, not external solver EOAs.
- **What we learned:** Solver deposits on-chain → Brale credits custodial → convert/mint → deliver → claim reimburses → custodial nets to zero.
- **Guide:** [brale-api.md](../guide/brale-api.md)
- `[learned: super-duper-intents/brain.md]`

## Revenue law in code

- **What we learned:** `checkedSettle` throws `UnprofitableFillError`; gate scripts exit non-zero; unprofitable quotes rejected at publish time.
- **Guide:** [testing.md §Gate scripts](../guide/testing.md)
- `[learned: super-duper-intents/plan.md, src/accounting/]`

## HMAC webhook → EIP-712 attestor hop

- **What happened:** Brale webhooks are HMAC; on-chain verifier needs ECDSA/EIP-712.
- **What we learned:** Attestor verifies HMAC, re-reads delivery on-chain, re-signs EIP-712 for `AttestationVerifier`.
- **Guide:** [brale-api.md §Webhooks](../guide/brale-api.md)
- `[learned: super-duper-intents/brain.md]`

## Capabilities fail-closed

- **What we learned:** Layers/assets/chains in YAML; env overrides (`CAP_LAYER2=1`); UI refuses orders solver can't fill before user locks funds.
- **Guide:** [getting-started.md](../guide/getting-started.md)
- `[learned: super-duper-intents/brain.md]`

## Poll don't subscribe on Workers

- **What we learned:** No WS on public RPCs in Workers; HTTP poll for EVM read-your-writes and Solana `getSignatureStatuses` with `searchTransactionHistory: true`.
- **Guide:** [cloudflare-stack.md](../guide/cloudflare-stack.md)
- `[learned: super-duper-intents/brain.md]`
