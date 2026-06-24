# brain.md

**Project source of truth.** Read this first. As-built only — not the plan doc.

**Last updated:** YYYY-MM-DD

---

## 1. What this is

One paragraph: mission, customer, current version/status.

## 2. Repo map

```
src/           …
packages/      …
```

## 3. Build & verify

```bash
pnpm install
pnpm -r typecheck
pnpm test
pnpm build
```

## 4. What's working

| Area | State | Notes |
|------|-------|-------|
| Feature A | ✅ | |
| Feature B | 🟡 | partial |

## 5. Known bugs / sharp edges

1. …
2. …

## 6. Conventions

- TypeScript strict
- …

## 7. Pointers

- [architecture.md](./architecture.md)
- [AGENTS.md](./AGENTS.md)
- [docs/decisions.md](./docs/decisions.md) (if present)

---

## Engineering log (append-only)

### YYYY-MM-DD — Title

What changed, why, root cause if bugfix.

---

## Lessons exported to the guide

| Date | Lesson | Guide location |
|------|--------|---------------|
| YYYY-MM-DD | Example lesson | guide/brale-api.md §Webhooks |

When you export a lesson to [super-duper-guide](https://github.com/superduperlabs/super-duper-guide), add a row here.
