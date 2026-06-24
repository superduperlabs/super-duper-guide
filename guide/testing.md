# Testing

Testing patterns across the Super Duper Apps family.

**Rule:** Tests are the deliverable — new features land with unit and/or E2E coverage.

`[learned: super-duper-wallet/AGENTS.md]`

## Unit tests (Vitest)

Pure logic, no browser, no network (mock RPC where needed).

| App | Scope |
|-----|-------|
| super-duper-wallet | `@sdw/core` — vault crypto, derivation, registry |
| super-duper-agent-wallet | `wallet.test.ts` — derivation, policy, destroy |

Run:

```bash
pnpm --filter @sdw/core test          # wallet
pnpm test                              # agent-wallet
```

## E2E (Playwright)

Load **built artifacts**, not mocks.

| App | Notes |
|-----|-------|
| super-duper-wallet | `@sdw/test`; headed Chrome; **`workers: 1`** (parallel headed runs flake) |
| super-duper-data | 70 responsive tests × 10 viewports; 47 security tests |

Wallet E2E caught CSP `wasm-unsafe-eval` silent failure — unit tests missed it.

`[learned: super-duper-wallet/AGENTS.md, ADR-006]`

Run:

```bash
pnpm --filter @sdw/extension build
pnpm --filter @sdw/test test:e2e      # wallet

npm run test:all                       # data (responsive + security)
```

## Gate scripts (CI-enforced)

Machine gates with non-zero exit on failure; persist logs.

**super-duper-intents:**

```bash
npm run gate:all        # all gates → test-results/
npm run test:revenue    # 15 assertions — unprofitable movement cannot settle
npm run gate:quote      # 25 assertions — quote parity + OIF conformance
```

Revenue law, quote floor, capabilities, sourcing, redemption — all enforced in code, not dashboards.

`[learned: super-duper-intents/brain.md, plan.md]`

## Real mainnet validation

Before claiming "works on mainnet", execute with small real funds:

| App | Validated flows |
|-----|-----------------|
| super-duper-wallet | SBC send Base, USDC send Solana, Uniswap V3, Jupiter |
| super-duper-intents | 41/41 routes; bidirectional Base ↔ Solana; same-chain buy/redeem |

Document results in `brain.md` and changelog.

## Security-specific tests

| App | Command |
|-----|---------|
| super-duper-data | `npm run test:security` |
| super-duper-wallet | Entropy regression test; CI grep for insecure RNG patterns |

## Quality gate order (wallet)

```bash
pnpm -r typecheck
pnpm --filter @sdw/core test
pnpm --filter @sdw/extension build
pnpm --filter @sdw/test test:e2e
```

A change is "done" only when all four are green.

`[learned: super-duper-wallet/AGENTS.md]`

## Regression discipline

Every bug fix → regression test. Record unfixed bugs in `brain.md` and `docs/status.md`.

`[learned: super-duper-wallet/CONTRIBUTING.md]`

## Related

- [security.md](security.md)
- [ai-agents.md](ai-agents.md) — what agents should run before claiming "done"
