---
name: env-config
description: Use when adding, renaming, or removing an environment variable in apps/web or apps/api, or when setting up local dev env files — covers NEXT_PUBLIC exposure rules, secret placement, and config validation.
---

# Environment/config conventions

## File layout

- Each app owns its own env files: `apps/web/.env.local` (gitignored) + `apps/web/.env.example` (committed), same pattern for `apps/api/`.
- Never share a single root `.env` across both apps — a var that's safe to expose from Next.js is not automatically safe in NestJS's context and vice versa.
- `.env.example` is kept in sync with every var actually referenced in code — `/env-check` audits this.

## The NEXT_PUBLIC rule

- Any env var read in `apps/web` client-side code **must** be prefixed `NEXT_PUBLIC_` and must not be a secret (API keys with write/delete scope, DB credentials, signing secrets never go here) — Next.js inlines these into the client bundle at build time.
- Anything read only in server components/Server Actions/route handlers does not need the prefix and should specifically avoid it if it's sensitive, to prevent accidental future client usage.

## Secret placement

- Database URL, JWT signing secret, third-party API secret keys live only in `apps/api/.env*` — never in `apps/web/.env*`, even unprefixed, since a misconfigured build step or an accidental client import could still leak it.
- If `apps/web` needs to call a third-party service that requires a secret key, that call goes through a NestJS endpoint (or a Next.js Server Action that reads a non-`NEXT_PUBLIC_` var), never directly from client code.

## Config validation

- Each app validates its env vars at startup against a zod schema (`apps/api/src/config/env.schema.ts`, `apps/web/lib/env.ts`) and fails fast with a clear error listing the missing/invalid var — don't let a missing var surface as a confusing runtime error three layers deep.
- Access env vars only through the validated config object (`config.databaseUrl`), not `process.env.DATABASE_URL` scattered through the codebase — this is what makes the zod validation actually load-bearing.

## Local bootstrap

1. Copy each app's `.env.example` to `.env.local` and fill in local values.
2. Start Postgres locally (Docker Compose service or local install).
3. Run `prisma migrate dev` in `apps/api` to bring the local DB schema up to date.
4. Run seed data per `postgres-database` if the project has a seed script.
