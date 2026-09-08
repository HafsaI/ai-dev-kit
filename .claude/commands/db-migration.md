---
description: Create and sanity-check a Prisma migration for the given change.
argument-hint: <description of the schema change>
---

Create a Prisma migration for: **$ARGUMENTS**

Follow `postgres-database`:

1. Edit `apps/api/prisma/schema.prisma` for the described change.
2. Run `prisma migrate dev --name <short-kebab-case-description>`.
3. Read the generated SQL in `prisma/migrations/<timestamp>_<name>/migration.sql`. If Prisma proposed a destructive step (drop+recreate) for what should be additive, stop and rewrite the migration by hand instead of applying it as-is.
4. Confirm any new/changed foreign key column has an index.
5. Run `prisma generate` if it didn't already run automatically.
6. Report back what changed, the migration name, and flag anything destructive for the user to confirm before it's committed.

If the change looks non-trivial or touches existing data (not just new tables/columns), delegate to the `db-migration-reviewer` agent for a second pass before finishing.
