---
description: Run once after copying this kit into a project — fills in project-context.md and reconciles the assumed layout against what actually exists.
argument-hint: (no arguments)
---

This kit was just copied into this project. Get it set up:

1. Ask the user (if not obvious from an existing README or package.json): project name, one-line description, current milestone/scope, and any domain terms worth recording in a glossary.
2. Inspect the actual repo structure (`ls`, check for `apps/web`, `apps/api`, `apps/api/prisma/schema.prisma`, `pnpm-workspace.yaml`/`turbo.json`, or a completely empty/greenfield repo).
3. Fill in `docs/project-context.md` with what you learned. If the actual layout differs from the kit's assumed `apps/web` + `apps/api` + `packages/shared` layout (e.g. different folder names, a single app, a different monorepo tool), record that under "Deviations from default kit conventions" — don't silently rewrite CLAUDE.md to match, since CLAUDE.md is meant to stay generic across projects.
4. If the project's database isn't plain PostgreSQL (e.g. Supabase), note it under deviations and tell the user they should swap `.claude/skills/postgres-database/SKILL.md` for a provider-specific version — don't attempt to rewrite that skill yourself unless asked.
5. Report back a short summary of what was filled in and any deviations recorded, so the user can correct anything before continuing.

Do not scaffold any application code in this step — this command only sets up context, it doesn't create apps/packages.
