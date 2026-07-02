# Contributing

Thanks for contributing to the **Super Duper Guide**.

## What this repo is

The shared playbook for the [Super Duper Apps](https://github.com/superduperlabs) family. It contains conventions, patterns, templates, and lessons learned across all apps.

## How to contribute

### Fix or improve a guide page

1. Edit the relevant file in `guide/`.
2. Keep the `[learned: repo/file]` provenance tags — they help trace where patterns came from.
3. Open a PR with a clear description of what changed and why.

### Add a lesson

1. Append to the relevant file in `lessons/` (one per app).
2. Use the format: date, title, what happened, what we learned.
3. If the lesson generalizes, also update the relevant `guide/` page.

### Add or update a template

1. Edit files in `templates/`.
2. Templates should be copy-paste ready — placeholders use `YYYY-MM-DD`, `{noun}`, etc.

## Pull requests

- Keep PRs focused.
- Verify links resolve (at least for public resources).
- Don't remove `[learned:]` tags — they're provenance, not clutter.

## Docs

- [guide/ai-agents.md](./guide/ai-agents.md) — how brain.md, AGENTS.md, and the learning loop work
- [guide/getting-started.md](./guide/getting-started.md) — anatomy of a Super Duper App

## License

By contributing, you agree your contributions are licensed under the project's [MIT license](./LICENSE).
