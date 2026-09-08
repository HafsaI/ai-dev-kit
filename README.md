# ai-dev-kit

A personal Claude Code dev kit for a fixed stack: **Next.js + NestJS + PostgreSQL (Prisma)**. It's not an app and has no runtime code — it's a set of Claude Code config (a root `CLAUDE.md`, `.claude/skills`, `.claude/agents`, `.claude/commands`, `.claude/settings.json`) meant to be **copied into every new project** you start.

## Usage

1. Copy `CLAUDE.md`, `docs/`, and `.claude/` from this repo into the root of your project (new or already-scaffolded — both work).
2. Start Claude Code in that project and run `/kit-init`. It asks a few questions, fills in `docs/project-context.md`, and reconciles the kit's assumed layout against whatever already exists.
3. Work as usual — Claude pulls in the relevant skill automatically when it's working on something that skill covers.

If a project's database isn't plain PostgreSQL (e.g. it uses Supabase instead), swap `.claude/skills/postgres-database/SKILL.md` for a provider-specific version in that copy of the kit, and note the deviation in `docs/project-context.md`. The rest of the kit doesn't need to change.

## What's in it

**Skills** (`.claude/skills/`) — on-demand reference playbooks, not always-loaded context:
- `nestjs-conventions`, `postgres-database`, `nextjs-conventions`, `api-contracts`, `testing-conventions`, `env-config`, `monorepo-tooling`

**Agents** (`.claude/agents/`) — isolated-context reviewers:
- `stack-code-reviewer` — read-only, staff-eng-style diff review
- `db-migration-reviewer` — read-only, focused on pending Prisma migrations

**Commands** (`.claude/commands/`):
- Scaffolding: `/kit-init`, `/new-feature`, `/db-migration`, `/api-endpoint`, `/env-check`
- Lifecycle: `/stack-test`, `/stack-review`, `/ship`

## Why no app scaffolding

This repo deliberately ships no `apps/web`, `apps/api`, or starter monorepo skeleton. It needs to work whether it's copied into a brand-new empty project or one that's already scaffolded, and vendored scaffold code (dependency versions, framework defaults) goes stale faster than conventions do. Use `create-next-app`, `nest new`, etc. directly, then copy this kit on top.

## Requirements

`.claude/settings.json` hooks shell out to `jq` to parse tool-call JSON — install it (`brew install jq` / `apt install jq`) or the format-on-write and migration-guard hooks silently no-op.

## Updating the kit later

`CLAUDE.md` and the `.claude/` files are meant to stay generic across projects — if you improve one, it's worth backporting into other projects' copies too. `docs/project-context.md` is the one file that's always project-specific and never gets backported.
