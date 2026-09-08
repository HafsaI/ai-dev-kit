---
name: postgres-database
description: Use when changing the Prisma schema, writing a migration, adding an index or foreign key, writing a non-trivial query, or seeding data for the PostgreSQL database in apps/api.
---

# PostgreSQL + Prisma conventions

> **Swap point:** if this project uses a different DB provider (e.g. Supabase instead of plain Postgres), this file has been replaced with a provider-specific version for that project — check `docs/project-context.md` for the noted deviation. What follows assumes plain PostgreSQL via Prisma.

## Schema conventions

- One `schema.prisma` at `apps/api/prisma/schema.prisma`, models in PascalCase singular (`User`, not `Users`), fields in camelCase.
- Every model has an `id String @id @default(cuid())` (or `@default(uuid())` if the project already standardized on UUIDs — don't mix within a project) and `createdAt`/`updatedAt` (`@default(now())` / `@updatedAt`).
- Every foreign key column gets an explicit `@@index` unless it's already covered by a `@@unique`/composite key — Postgres does not auto-index FK columns.
- Prefer `onDelete: Restrict` by default; only use `Cascade` where losing child rows on parent delete is the actual intended behavior, and say so in a comment.

## Migration workflow

1. Edit `schema.prisma`.
2. Run `prisma migrate dev --name <description>` — this generates the SQL migration under `prisma/migrations/` **and** applies it to the local dev DB.
3. Read the generated SQL before moving on. Prisma sometimes proposes a destructive step (drop + recreate a column) for changes that could be additive — rewrite the migration by hand if so.
4. Never hand-edit an already-applied migration file; create a new one instead.
5. Run `prisma generate` (usually automatic after `migrate dev`) so the Prisma Client types stay in sync — do this before touching any service code that uses the changed model.
6. For production, migrations apply via `prisma migrate deploy` in CI/CD, not `migrate dev` — deploy-time application always needs explicit confirmation (see `.claude/settings.json`).

## Query patterns

- All DB access goes through `PrismaService` (a thin wrapper extending `PrismaClient`, provided as a Nest injectable) — services depend on it via constructor injection, never instantiate `PrismaClient` directly.
- Use `select`/`include` explicitly for anything returned across the API boundary; don't return a full Prisma model with fields (e.g. password hashes, internal flags) that shouldn't leave the service layer.
- Batch related writes in `prisma.$transaction([...])` when they must succeed or fail together.
- For pagination, use `skip`/`take` with a stable `orderBy` (`id` or `createdAt`); avoid `cursor` pagination unless a feature specifically needs stable pagination under concurrent inserts.

## Seeding

- `apps/api/prisma/seed.ts`, run via `prisma db seed`. Seed data should be idempotent (`upsert`, not `create`) so it can be re-run safely against a dev DB that already has data.
