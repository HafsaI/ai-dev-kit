---
name: monorepo-tooling
description: Use when running dev/build/test scripts, adding a new package or app to the workspace, or changing shared lint/format/tsconfig — covers the turborepo/pnpm workspace layout and task graph.
---

# Monorepo tooling conventions

## Workspace layout

pnpm workspaces + Turborepo:

```
apps/
  web/       Next.js — package name @app/web
  api/       NestJS — package name @app/api
packages/
  shared/    Shared zod schemas/types — package name @app/shared
  ui/        Shared React components (if/when frontend-only code needs sharing beyond apps/web)
pnpm-workspace.yaml
turbo.json
package.json     root — dev tooling only, no app dependencies
```

`apps/*` depend on `packages/shared` via the workspace protocol (`"@app/shared": "workspace:*"`), never by relative path across the app boundary.

## Common commands

- `pnpm install` — installs for the whole workspace from the root; never `cd` into an app and run `npm install` there.
- `pnpm turbo dev` — runs each app's `dev` script in parallel with the right task graph (so `packages/shared` builds/watches before the apps that consume it).
- `pnpm turbo build` — full production build, respecting `dependsOn` in `turbo.json` so `packages/*` build before `apps/*`.
- `pnpm turbo lint` / `pnpm turbo test` — run across every workspace package; scope to one with `pnpm turbo test --filter=@app/api`.

## turbo.json task graph

- Each task that depends on another package's build output declares `"dependsOn": ["^build"]` — this is what makes `packages/shared`'s types available before `apps/web`/`apps/api` type-check against them.
- Cache `build`/`lint`/`test` outputs (`"outputs": [...]`) but never cache `dev` (`"cache": false`, `"persistent": true`).

## Shared config

- One root `tsconfig.base.json` that each app/package's `tsconfig.json` extends — don't duplicate `compilerOptions` per package.
- One root ESLint/Prettier config, extended (not overridden) per package for framework-specific rules (e.g. Next.js's ESLint plugin only in `apps/web`).

## Adding a new package

1. Create `packages/<name>/package.json` with `"name": "@app/<name>"`.
2. Add it to `pnpm-workspace.yaml` if the glob (`apps/*`, `packages/*`) doesn't already cover it — usually it does.
3. Add `"@app/<name>": "workspace:*"` to any app that needs it, then `pnpm install` from root.
