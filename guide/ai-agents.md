# Working with AI agents

How to structure repos so humans and AI agents collaborate effectively.

## Document layers

| File | Purpose | Best example |
|------|---------|--------------|
| **`brain.md`** | As-built truth: what's shipped, where things live, bugs, next steps | super-duper-wallet |
| **`AGENTS.md`** | Working agreement: build order, conventions, guardrails | super-duper-wallet |
| **`architecture.md`** | Runtime architecture + file index | super-duper-wallet |
| **`docs/decisions.md`** | Append-only ADR log (Decision · Rationale · Consequences) | super-duper-wallet ADR-001–014 |
| **`docs/status.md`** | Feature matrix + known bugs | super-duper-wallet |

Read **`brain.md` first**, then the relevant architecture section, then `AGENTS.md` before making changes.

## brain.md conventions

- **As-built, not aspirational** — if it's not in the repo, don't claim it's done
- **Append-only** dated entries for engineering log (intents uses this heavily)
- **Lessons exported** section at bottom — tracks what's been shared to super-duper-guide
- Update when behavior changes

## AGENTS.md conventions

Every app should have AGENTS.md (wallet is the template). Include:

1. Build & verify commands (in order)
2. Code conventions (TypeScript strict, crypto libs, comments)
3. **Recursive learning** section — link to super-duper-guide and brale-agent-kit
4. "Don't commit unless explicitly asked"

Copy from [templates/AGENTS.md](../templates/AGENTS.md).

## Brale Agent Kit

For Brale API work, agents should also read:

- [superduperdot/brale-agent-kit/AGENTS.md](https://github.com/superduperdot/brale-agent-kit/blob/main/AGENTS.md)
- `[learned: repo/file]` tags trace every non-obvious rule to source

When discovering a Brale API pitfall, update **both** the app's brain.md and brale-agent-kit when appropriate.

## ADR discipline

Write ADRs when:

- Reversing a prior decision (wallet ADR-014 gasless revert is the canonical example)
- Choosing between architectures with long-term cost
- Documenting a sharp edge that will bite the next agent

Keep superseded ADRs with strikethrough notes — don't delete history.

## experiments/

Throwaway spikes live outside main packages.

**Where this is used:** super-duper-wallet `experiments/` (passkey-prf, wallet-discovery)

## Recursive learning loop

When an agent learns something reusable:

1. Add to this repo's `brain.md` (dated)
2. If it helps other apps → `super-duper-guide/lessons/<app>.md` + relevant `guide/*.md`
3. Tag: `[learned: <repo>/<file>]`
4. Update **Lessons exported** table in brain.md

See [lessons/README.md](../lessons/README.md) for trigger list.

## Agent guardrails (wallet family)

- Don't add custodial vendors or chains beyond scoped set without explicit approval
- Provider trust: derive origin from `sender.origin`, never page-supplied
- Record new bugs in status.md if not fixing now

`[learned: super-duper-wallet/AGENTS.md]`

## Related

- [templates/brain.md](../templates/brain.md)
- [templates/AGENTS.md](../templates/AGENTS.md)
- [getting-started.md](getting-started.md)
