---
description: Scaffold a single API endpoint end-to-end — NestJS controller method + DTO, shared schema, and a typed Next.js client function.
argument-hint: <method> <path> <description>
---

Scaffold one endpoint: **$ARGUMENTS** (expected form: `<HTTP method> <path> <description>`, e.g. `POST /users/:id/invite send an invite email to an existing user`)

Per `api-contracts` and `nestjs-conventions`:

1. Add/extend the zod schema in `packages/shared/src/schemas/` for this endpoint's request and response shape, if one doesn't already exist for this feature.
2. Add the controller method (thin — validate via DTO, delegate to the service) and the service method implementing the actual logic.
3. Ensure the response follows the standard envelope (`{ data }`, `{ data, meta }`, or the error shape) — don't invent a new shape.
4. Add a typed client function in `apps/web/lib/api/<feature>.ts` that calls this endpoint and parses the response through the shared schema.
5. Add a controller test for request validation and a service test for the main logic path, per `testing-conventions`.

Report the final route, request/response shape, and where each piece landed.
