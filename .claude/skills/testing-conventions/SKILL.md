---
name: testing-conventions
description: Use when writing or reviewing tests for NestJS services/controllers, Next.js components/pages, or end-to-end flows across the stack — covers tooling, file naming, and what to test where.
---

# Testing conventions

## Where each kind of test lives

| Layer | Tool | File pattern | What it covers |
|---|---|---|---|
| NestJS services | Jest | `*.service.spec.ts` | Business logic, mocking `PrismaService` |
| NestJS controllers | Jest | `*.controller.spec.ts` | Request validation/routing, mocking the service |
| NestJS e2e | Jest + Supertest | `test/*.e2e-spec.ts` | Full HTTP request through a real (test) DB |
| Next.js components | Vitest + React Testing Library | `*.test.tsx` | Rendering, interaction, accessibility roles |
| Cross-app e2e | Playwright | `e2e/*.spec.ts` | Real browser against a running web+api+DB stack |

Don't reach for Playwright to test something a component test or a Nest e2e test already covers cheaper and faster.

## NestJS unit tests

- Mock `PrismaService` (or any external dependency) at the constructor-injection boundary using Nest's `Test.createTestingModule` — don't hit a real database in `.spec.ts` unit tests.
- Test the service's business logic (edge cases, error throwing) directly; test the controller only for request-shape concerns (DTO validation, correct service method called) since business logic is already covered in the service test.

## NestJS e2e tests

- Run against a real Postgres instance (a disposable test DB, reset between runs via `prisma migrate reset --force` or a transaction-rollback pattern) — never mock Prisma here, the point is to catch real query/migration issues.
- Cover the full request → validation → service → DB → response round trip for each feature's critical paths (create, list, not-found, validation failure, auth failure).

## Next.js component tests

- Test server components by rendering their output where feasible; for client components, test user-visible behavior (what renders, what happens on interaction) — not implementation details like internal state variable names.
- Mock the API client layer (`apps/web/lib/api/*`) rather than mocking `fetch` directly, since that's the actual seam the app code depends on.

## Playwright e2e

- Reserve for the handful of true cross-app critical paths (signup → login → core action → logout). These are the slowest and flakiest tests — keep the count small and deliberate.
- Run against a docker-composed or locally-started full stack, seeded via the `postgres-database` skill's seed script.

## Naming and structure

- Test file sits next to the file it tests (co-located), except Playwright e2e which lives in a top-level `e2e/` directory since it doesn't correspond to one source file.
- Test descriptions read as a sentence: `describe('UsersService', () => { it('throws NotFoundException when the user does not exist', ...) })`.
