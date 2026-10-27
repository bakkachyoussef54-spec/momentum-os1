# Vercel migration guide — work required, not an already-applied conversion

## Why the exact export cannot be deployed as-is

The repository contains Next.js-style App Router code, but its current scripts build **Vinext for Cloudflare Workers**. Standard Next.js hosting on Vercel does not supply this project's `cloudflare:workers` module, D1 `env.DB` binding or Sites identity gateway. The UI can be retained while adapting those backend/runtime boundaries.

This document identifies the necessary replacements. No migration code or secrets have been inserted into the original application, and no Vercel deployment was attempted.

## Dependency replacement map

| Existing boundary | Source files | Required independent replacement |
| --- | --- | --- |
| Vinext/Cloudflare build and Worker entry | `package.json`, `vite.config.ts`, `worker/index.ts`, `scripts/run-framework.mjs` | Standard Next.js App Router build/runtime for Vercel; compatible dependency versions and regenerated lockfile in a separate migration branch |
| `env.DB`, D1 SQL prepare/bind/batch | `app/api/data/route.ts`, `db/index.ts` | Server-side database client for an independently owned database; preserve parameterized queries, transactions, uniqueness and optimistic versions |
| SQLite schema | `db/schema.ts`, `drizzle/0000_crazy_wrecker.sql` | Equivalent migrations for the selected database; SQLite migration syntax is not directly PostgreSQL SQL |
| Trusted `oai-authenticated-user-*` headers | `app/chatgpt-auth.ts` and API callers | Verified server-side authentication/session provider; derive owner from the verified session, never an arbitrary request header |
| Gateway-owned sign-in/sign-out paths | `app/chatgpt-auth.ts`, hardcoded links in `app/workspace.tsx` | Real auth callbacks, login, logout and session refresh routes for the selected provider |
| Site project identity and private access | `.openai/hosting.json` | Vercel project configuration plus application-level access control; do not carry over Sites deployment automation |
| Optional connector infrastructure | `lib/connectors.ts`, `lib/connector-*`, `components/connector-error.tsx`, Worker wrapper and Vite connector plugin | Remove unused helpers from the migrated build or implement independent integrations if ever used. Current Momentum UI does not call these connectors |
| Fonts and icons | `app/globals.css`, dependency lockfile | Preserve current Google Fonts CSS and Lucide dependencies, or separately license/self-host fonts if desired |

## Migration sequence

1. Keep this ZIP as your untouched reference. Work in a separate copy or Git branch.
2. Preserve `app/workspace.tsx`, `app/globals.css`, assets and domain behavior. Change only auth link wiring where required; verify screenshots before and after.
3. Configure a supported, mutually compatible Next.js/React/TypeScript toolchain. Replace `dev`, `build` and `start` with `next dev`, `next build` and `next start`. Remove Vinext type augmentation and Cloudflare-only type/build configuration from the migrated project. The original pins are retained in this ZIP; compatibility must be tested, not assumed.
4. Choose independent auth and storage. One possible combination is Supabase Auth plus PostgreSQL. Other services are possible, but their setup and keys differ. Obtain accounts/credentials directly through the chosen provider; none are included here.
5. Implement a verified server-side session helper and protect every API route. Replace the original Sites header trust and auth links. Do not solve login by trusting a user ID from the browser or disabling authorization.
6. Port the records table and database calls. Retain `id`, `owner`, `kind`, `data`, `version`, timestamps and nullable `natural_key`; preserve indexes on owner/kind and uniqueness on owner/natural_key. Keep JSON serialization behavior compatible with the API response. Convert SQLite-specific operations such as `INSERT OR IGNORE` to the selected database's conflict syntax. Execute related deletes/updates atomically. Preserve the single-active-timer constraint and stale-version rejection.
7. Keep `/api/data` request/response behavior compatible so the existing frontend continues working. If using Supabase directly, use user-session authorization/RLS or equivalent verified-owner enforcement; never expose a service-role credential in browser variables.
8. Export your actual live records through Settings → Export all data (JSON). Implement a reviewed import process for the new database. The original source has no data-import endpoint. Map ownership to the new verified account ID while retaining record IDs and relationships; calculate uniqueness keys consistently. Back up before importing. Source export alone does not copy your hosted data.
9. Run unit tests, Next.js production build and authenticated end-to-end tests before deploying production.

## Environment variables after migration

The original export consumes no `DATABASE_URL` or Supabase variables. Adding these names alone will not make it portable; the replacement adapters must read them.

For a Supabase-based migration, commonly used names are `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`, plus any server-only database connection string required by your chosen database adapter. Match the exact names used by the implemented integration. Public publishable keys require appropriate database access policies; private database/service-role credentials must remain server-only.

Create `.env.local` on your computer and set equivalent values in Vercel Project Settings → Environment Variables for Development, Preview and Production as appropriate. Do not commit `.env.local`. Redeploy after changing hosted variables. Configure exact login callback/redirect URLs for localhost and the new hosted domain in the auth provider.

AI Coach remains deferred. No OpenAI API key is needed for existing functionality.

## Deploy to Vercel — after the migration passes

1. Push the migrated copy to a repository you own.
2. Import that repository in Vercel. Set Root Directory to the folder containing its `package.json`.
3. Select the Next.js framework preset. Use the migrated standard build script and framework-default output; do not point Vercel at the original Worker `dist/server` directory.
4. Add the provider-specific environment variables and configure callback URLs.
5. Apply the new database migrations using your selected provider's documented migration workflow. Do not execute the original SQLite migration against PostgreSQL unchanged.
6. Create a Preview deployment and test registration/login/logout, private routes, goal/task CRUD, reload persistence, owner isolation, recurring deduplication, habits, timer refresh/pause/resume, analytics, income, exports and deletion.
7. Compare the migrated design at desktop and mobile sizes against the existing Site, then promote a verified deployment to production.

Vercel must connect to durable external database storage; do not use an ephemeral function filesystem as your production database. The original Sites sign-in and D1 access are not transferable credentials.

## Completion criteria

Independent hosting is complete only when the migrated app runs without `cloudflare:workers` imports, Sites-injected identity headers, gateway auth routes or Sites-managed database bindings; its new auth and database are configured and tested; the UI matches; and a real Vercel deployment passes authenticated smoke tests. This export alone does not claim those steps are complete.

## Official references

- Next.js on Vercel: https://vercel.com/docs/frameworks/full-stack/nextjs
- Vercel environment variables: https://vercel.com/docs/environment-variables
- Environment-variable management: https://vercel.com/docs/environment-variables/managing-environment-variables
- D1 local development (original runtime): https://developers.cloudflare.com/d1/best-practices/local-development/

References checked 2026-10-27. Provider-specific implementation details should be verified when performing the actual migration.
