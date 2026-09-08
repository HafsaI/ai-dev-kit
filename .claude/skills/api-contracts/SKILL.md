---
name: api-contracts
description: Use when defining or changing an endpoint's request/response shape, adding a shared type used by both apps/web and apps/api, or wiring a new Next.js client call to a NestJS endpoint.
---

# API contract conventions

## Single source of truth

- Request/response shapes are defined once as zod schemas in `packages/shared/src/schemas/<feature>.ts`, imported by both apps — never redefined independently on each side.
- NestJS validates incoming requests against the same zod schema (via `nestjs-zod` or a thin pipe wrapping `schema.parse`), in addition to (not instead of) the `class-validator` DTOs used for Swagger/OpenAPI generation — the zod schema is the cross-app contract, the DTO is Nest's internal validation mechanism.
- Next.js uses the same schema client-side for form validation and to type the response of its API client functions.

## Response envelope

Single resource:
```ts
{ data: T }
```

Paginated list:
```ts
{ data: T[], meta: { page: number, pageSize: number, total: number } }
```

Error (set by the global `HttpExceptionFilter` in `nestjs-conventions`):
```ts
{ error: { code: string, message: string, details?: unknown } }
```

Every endpoint returns one of these three shapes — no bare arrays, no ad hoc top-level fields.

## Error codes

- `code` is a stable, machine-readable string (`VALIDATION_ERROR`, `NOT_FOUND`, `UNAUTHORIZED`, `CONFLICT`, feature-specific codes like `EMAIL_ALREADY_EXISTS`) that the frontend can switch on — never rely on parsing `message` for control flow.
- `message` is human-readable and safe to show to a user; never leak stack traces or internal identifiers into it.

## Versioning

- Prefix routes with `/api/v1` from day one. Only introduce `/api/v2` for a genuinely breaking change to an existing endpoint's contract — additive fields don't need a new version.

## Pagination/filtering

- List endpoints accept `page` (1-indexed) and `pageSize` query params (default `pageSize=20`, cap at a sane max like 100 server-side regardless of what's requested).
- Filters are individual query params (`?status=active`), not a single opaque JSON blob param.

## Client wiring

- Next.js calls the API through a small typed client in `apps/web/lib/api/<feature>.ts` — one function per endpoint, parsing the response through the shared zod schema before returning it to the caller, so a contract drift fails fast at the call site instead of silently passing through malformed data.
