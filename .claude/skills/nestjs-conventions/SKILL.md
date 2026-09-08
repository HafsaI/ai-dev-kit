---
name: nestjs-conventions
description: Use when creating or editing a NestJS module, controller, service, DTO, guard, or exception filter in apps/api — covers folder structure, dependency injection, validation, auth guards, and response shape conventions for this stack.
---

# NestJS conventions

## Folder structure

One folder per domain feature under `apps/api/src/`, not per technical layer:

```
apps/api/src/
  users/
    users.module.ts
    users.controller.ts
    users.service.ts
    dto/
      create-user.dto.ts
      update-user.dto.ts
    users.controller.spec.ts
    users.service.spec.ts
```

Cross-cutting infra (guards, filters, interceptors, the Prisma module) lives in `apps/api/src/common/`.

## Module/controller/service

- One `@Module` per feature, exporting only what other modules actually import.
- Controllers stay thin: validate input (via DTO), call one service method, return its result. No business logic in controllers.
- Services own business logic and are the only layer that talks to Prisma (`postgres-database` skill). Never inject `PrismaService` into a controller.
- Use constructor injection everywhere; avoid property injection.

## DTOs and validation

- Every controller method that accepts a body/query uses a `class-validator`-decorated DTO — never an untyped `any`/`object` param.
- Global `ValidationPipe` (in `main.ts`) with `whitelist: true, forbidNonWhitelisted: true, transform: true` — reject unknown fields rather than silently dropping them.
- DTOs are the request-shape source of truth; response shapes come from `packages/shared` per `api-contracts`, not from a separate "response DTO" class.

## Auth guard

Standard NestJS Passport/JWT setup unless `docs/project-context.md` says otherwise:

- `JwtStrategy` (via `passport-jwt`) validates the access token and attaches the user to `request.user`.
- A global `JwtAuthGuard` applied via `APP_GUARD`, with `@Public()` decorator (custom metadata + guard check) for the few unauthenticated routes (login, signup, health check).
- Role/permission checks are a separate `RolesGuard` reading a `@Roles(...)` decorator — don't fold authorization logic into the auth guard itself.

## Errors and response shape

- Throw Nest's built-in HTTP exceptions (`NotFoundException`, `BadRequestException`, etc.) from services; don't return error objects with a 200 status.
- A global `HttpExceptionFilter` normalizes every error response to the shape defined in `api-contracts` (`{ error: { code, message, details? } }`) — don't hand-roll error bodies per-endpoint.
- Successful responses follow the shared envelope from `api-contracts` (plain payload for single resources, `{ data, meta: { page, pageSize, total } }` for paginated lists).

## Templates

`templates/` in this skill has starter files (`module.ts.template`, `controller.ts.template`, `service.ts.template`, `dto.ts.template` — `.ts.template` so they don't get parsed as live TypeScript) — copy, strip the `.template` suffix, and find-and-replace the placeholder word `Feature`/`feature` throughout (filenames included, e.g. `feature.service.ts` → `users.service.ts`), rather than writing a module from scratch. `/new-feature` uses these directly. Placeholder conventions (no underscores — plain words, replaced by case):

- `Feature` → PascalCase feature name (e.g. `User`)
- `feature` → camelCase feature name (e.g. `user`), including the Prisma model accessor (`this.prisma.feature` → `this.prisma.user`)
- `feature-plural` → kebab-case plural for the route path (e.g. `users`)
