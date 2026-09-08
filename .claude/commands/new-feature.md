---
description: Scaffold a full vertical slice for a new feature — Prisma migration, NestJS module, Next.js route, shared schema, and tests.
argument-hint: <feature-name>
---

Scaffold a complete vertical slice for the feature: **$ARGUMENTS**

Follow this order, consulting each skill as you reach its step:

1. **Shared schema** (`api-contracts`) — define the zod schema(s) for this feature's request/response shapes in `packages/shared/src/schemas/`.
2. **Database** (`postgres-database`) — add the Prisma model, run `prisma migrate dev --name <feature>`, verify the generated SQL and indexes.
3. **NestJS** (`nestjs-conventions`) — create the module/controller/service/DTOs using the templates in that skill's `templates/` directory as a starting point; wire validation, and reuse the shared zod schema alongside the DTO per `api-contracts`.
4. **Next.js** (`nextjs-conventions`) — add the route/page, a typed API client function (per `api-contracts`), and a Server Action for any mutation.
5. **Tests** (`testing-conventions`) — service unit test (main path + one failure case), controller test for request validation, and a component test for any new non-trivial Next.js UI.

After scaffolding, run `/stack-test` to confirm everything passes before considering the feature done. Ask the user before making any judgment call not covered by the skills (e.g. feature-specific business rules) rather than guessing.
