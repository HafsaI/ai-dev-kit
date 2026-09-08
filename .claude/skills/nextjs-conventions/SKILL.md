---
name: nextjs-conventions
description: Use when adding or editing a route, page, layout, component, form, or data fetch in apps/web — covers App Router file conventions, server vs client component boundaries, Server Actions, and revalidation.
---

# Next.js conventions

## App Router file structure

Feature-folder co-location under `apps/web/app/`, not a shared `components/` dump for feature-specific UI:

```
apps/web/app/
  (marketing)/            route group, no URL segment
  dashboard/
    layout.tsx
    page.tsx
    _components/          feature-local components (leading underscore = not a route)
    actions.ts            Server Actions for this route
  api/                    only for webhooks/third-party callbacks — app-to-app calls go through NestJS, not Next API routes
```

Truly shared, cross-feature UI (buttons, inputs, layout primitives) lives in `apps/web/components/` or `packages/ui` if shared with other apps.

## Server vs. client components

- Default to server components. Add `'use client'` only when a file needs interactivity (state, effects, browser APIs, event handlers) or a client-only library.
- Push `'use client'` as far down the tree as possible — wrap just the interactive leaf, not the whole page, so the rest stays server-rendered.
- Never import server-only code (Prisma client, secrets, server SDKs) into a file that has or is imported by a `'use client'` boundary.

## Data fetching and mutations

- Reads: fetch data directly in server components/layouts (`await` inside the component), not via `useEffect` + client fetch, unless the data genuinely depends on client-side state.
- Writes: use Server Actions (`'use server'` functions, typically in a route's `actions.ts`) for form submissions and mutations instead of a client-side `fetch` to a Next API route.
- Server Actions call the NestJS API (never Prisma directly from `apps/web` — the API boundary is NestJS) and validate input with the same shared zod schema from `packages/shared` used on the NestJS DTO side (see `api-contracts`).
- After a mutation, call `revalidatePath`/`revalidateTag` for the affected data rather than a full page reload or client-side refetch loop.

## Forms

- Use the shared zod schema (from `packages/shared`) for both client-side validation (e.g. via `react-hook-form` + `zodResolver`) and re-validate the same schema server-side in the Server Action — never trust client validation alone.
- Surface Server Action errors via return value (`{ error: ... }`), not thrown exceptions across the server/client boundary.

## Auth

- Session/token handling in Next.js reads the access token issued by the NestJS auth endpoints (see `nestjs-conventions`) and attaches it as an `Authorization` header on server-side calls to the API — don't duplicate auth logic in Next.js beyond storing/forwarding the token.
