---
description: Audit environment variables for unused entries, missing entries, and accidentally-exposed secrets.
argument-hint: (no arguments)
---

Audit env var hygiene per `env-config`:

1. For each app (`apps/web`, `apps/api`), grep source for every `process.env.*` / config-object reference and compare against that app's `.env.example`.
2. Flag: vars referenced in code but missing from `.env.example`; vars in `.env.example` no longer referenced anywhere; any `NEXT_PUBLIC_*` var in `apps/web` whose name or apparent purpose suggests it's a secret (contains `secret`, `private`, `service_role`, `key` combined with write/delete-scoped context, etc.) — these need a second look even if the code "works," since a leaked secret is a security bug regardless of whether anything currently depends on it being secret.
3. Confirm neither app's `.env.example` contains a real-looking value (an actual key/URL rather than a placeholder) — flag if so, since example files are committed.
4. Report findings grouped by category (missing / unused / possibly-leaked / real-value-in-example). Don't edit any `.env*` file yourself — env changes are the user's call.
