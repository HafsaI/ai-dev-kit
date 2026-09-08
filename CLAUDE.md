# ai-dev-kit

This is a personal Claude Code dev kit, copied wholesale (`.claude/`, this file, `docs/`) into the root of each new project. It codifies conventions for a fixed stack: Next.js, NestJS, PostgreSQL. Detailed playbooks live in `.claude/skills/` and load on demand — this file only holds what must always be in context.

@docs/project-context.md

## Stack

- **Frontend:** Next.js, App Router.
- **Backend:** NestJS.
- **Database:** PostgreSQL via Prisma. If this project uses a different provider (e.g. Supabase), `postgres-database` in `.claude/skills/` has been swapped for a provider-specific version — check `docs/project-context.md` for any noted deviation.

## Assumed layout

Convention, not a hard requirement — `/kit-init` reconciles this against whatever actually exists:

```
apps/
  web/       Next.js app
  api/       NestJS app
packages/
  shared/    Shared types/schemas (zod) consumed by both apps
```

## Non-negotiable rules

- Schema changes only via committed Prisma migrations (`prisma migrate dev`) — never manual/dashboard DB edits.
- Secrets never in client-exposed code. Only `NEXT_PUBLIC_*`-prefixed env vars may reach the browser; anything else stays server-only.
- API responses follow the shared contract shape defined in `packages/shared` — no ad hoc response shapes.
- No unchecked `any` at an API boundary (NestJS DTO in, zod-validated response out).
- Auth uses standard NestJS Passport/JWT guards unless `docs/project-context.md` states otherwise.
- Destructive git operations (force-push, hard reset) and pushing/opening PRs always require explicit user confirmation first.

## Skills (loaded on demand — see each SKILL.md for exact trigger phrasing)

| Concern | Skill |
|---|---|
| NestJS module/controller/service/DTO structure, auth guards | `nestjs-conventions` |
| Prisma schema, migrations, indexing, query patterns | `postgres-database` |
| Next.js App Router, server/client components, Server Actions | `nextjs-conventions` |
| Shared request/response contracts between web and api | `api-contracts` |
| Test structure and tooling (Jest, Vitest, Playwright) | `testing-conventions` |
| Env var conventions and secret placement | `env-config` |
| Monorepo/workspace/task-runner layout | `monorepo-tooling` |

## Commands

**Scaffolding/reference**
- `/kit-init` — run once after copying this kit into a project; fills `docs/project-context.md` and reconciles against existing scaffold.
- `/new-feature <name>` — scaffolds a full vertical slice (migration, Nest module, Next route, shared schema, tests).
- `/db-migration <description>` — creates and sanity-checks a Prisma migration.
- `/api-endpoint <method> <path> <description>` — scaffolds one endpoint end-to-end.
- `/env-check` — audits env vars for unused/missing/leaked-secret issues.

**Lifecycle**
- `/stack-test` — runs the relevant test suites for the current change and summarizes failures.
- `/stack-review` — staff-eng-style review of the current diff via the `stack-code-reviewer` agent.
- `/ship` — tests + lint/typecheck + drafts a PR description; stops for confirmation before pushing or opening the PR.

## Dev workflow

Day-to-day commands (`turbo dev`, `prisma studio`, etc.) and workspace task-graph conventions are documented in `monorepo-tooling` — consult it rather than guessing script names.
