---
description: Run the relevant test suites for the current change and summarize failures.
argument-hint: [optional scope, e.g. "apps/api" or a feature name]
---

Run tests for the current change ($ARGUMENTS, or infer scope from `git diff` if no argument given), per `testing-conventions`:

1. Determine which apps/packages the current diff touches.
2. Run the appropriate suites: `pnpm turbo test --filter=@app/api` for NestJS changes (Jest unit + e2e if the DB/test env is available), `pnpm turbo test --filter=@app/web` for Next.js changes (Vitest), Playwright only if explicitly asked or the change touches a cross-app critical path.
3. If a test fails because the change under test is genuinely broken, fix the source, not the test.
4. If a test fails because it's outdated relative to an intentional behavior change, update the test — but say explicitly which failures you treated this way and why, don't silently loosen assertions.
5. Report a summary: suites run, pass/fail counts, and full detail on any failure that's still unresolved.
