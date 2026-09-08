---
description: Test + lint/typecheck + draft a PR description for the current change — stops for confirmation before pushing or opening the PR.
argument-hint: (no arguments)
---

Prepare the current change to ship:

1. Run `/stack-test` (or its underlying steps) — do not proceed if there are unresolved failures.
2. Run `pnpm turbo lint` and `pnpm turbo typecheck` (or `tsc --noEmit` per app if no dedicated typecheck script exists) across affected packages; fix anything that fails.
3. If the diff touches `prisma/migrations/`, run the `db-migration-reviewer` agent and resolve anything it flags before continuing.
4. Run `/stack-review` and address findings, or explicitly note which ones you're deliberately not addressing and why.
5. Draft a PR title (under ~70 characters) and a short description (what changed and why, plus a test plan checklist), based on the actual commits/diff on this branch.
6. **Stop here and show the user the draft PR title/description.** Do not stage, commit, push, or open a PR until they explicitly confirm — pushing and PR creation are user-confirmed actions per this project's standing rules, not something to do autonomously.
7. Once confirmed, proceed with committing (if needed), pushing, and opening the PR via `gh pr create`.
