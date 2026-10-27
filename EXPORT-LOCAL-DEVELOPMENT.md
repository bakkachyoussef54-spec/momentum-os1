# Local development of the original project

These instructions preserve the exported source and use its existing portable development mode. They do not connect to or alter the hosted Site database.

## 1. Install dependencies

Unzip the archive and enter the `momentum-os` folder. Use a supported Node.js release satisfying the original `>=22.13.0` engine and the requirements of the pinned package manager. The project records `pnpm@11.25.0`; retain that version and the lockfile for reproducibility.

```sh
cd momentum-os
npm install --global pnpm@11.25.0
pnpm install --frozen-lockfile
```

If that exact package-manager/dependency version is unavailable from your registry or requires a newer Node release, installation is blocked until the matching toolchain is available. Do not silently replace the lockfile and call the result the original version. Use `pnpm install`, not the managed-container `install:ci` helper.

## 2. Environment configuration

The current release requires no AI or database passwords locally. `.env.example` documents this. You can copy it to `.env.local` for future configuration; do not put real secrets in shared source control.

The original `.openai/hosting.json` declares `d1: "DB"`. `vite.config.ts` supplies the local D1 binding configuration. These are runtime bindings, not environment variables. `DATABASE_URL` by itself has no effect on this original code.

Do not copy `.sites-runtime` or `.wrangler` from an existing hosted/managed checkout. They are intentionally omitted; a clean export chooses the portable profile. Google Fonts require network access for the original typefaces; a later offline adaptation would need licensed local font assets.

## 3. Build and initialize the local database

```sh
pnpm run build
node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0000_crazy_wrecker.sql
```

Build generates the local Wrangler configuration. Apply this migration once to a fresh local database. Do not repeatedly apply its CREATE TABLE statements to the same database. Future schema changes use `pnpm run db:generate`, followed by the newly generated migrations in order.

This database is local only and starts empty. No production backup or data import occurs automatically. Do not replace `--local` with `--remote` for these instructions.

## 4. Start development and sign in locally

```sh
pnpm run dev
```

The portable script uses port 5173 by default. Open the loopback URL printed by the dev server, normally `http://localhost:5173`.

Use the app's sign-in link, or visit:

```text
http://localhost:5173/signin-with-chatgpt?return_to=/
```

The bundled Vite development plugin simulates identity for loopback-only requests using a local test user (`local_seedy`). This is **not** production authentication and does not require your real ChatGPT credentials. It does not sign in to the hosted website. The local session persists in a browser cookie. Requests through non-loopback hosts cannot use this mock sign-in.

Create a test goal/task, refresh, and verify it remains. Start a focus session and navigate away/back. These are useful local smoke tests; they have not been newly run against this downloaded copy.

`pnpm run start` previews the built Worker via Wrangler, but it does not provide the portable Vite sign-in simulation. Use `pnpm run dev` for the signed-in local UI.

## 5. Checks

```sh
pnpm exec tsc --noEmit
pnpm exec eslint app db tests --no-warn-ignored
node --experimental-strip-types --test tests/domain.test.ts
pnpm run build
```

`next-env.d.ts` is included as generated in the existing checkout and refers to generated framework types. Build/development regenerates framework state; do not copy a managed `.next` cache.

`tests/api-integration.py` is the original managed-preview test script and targets `terminal.local:4173` with synthetic identity headers. It is not a ready-made test of portable login or public hosting, where those headers must not be trusted. Preserve it as source; adapt a separate test copy to authenticated sessions for a new hosting environment. Its success was not established during the original build.

## Source map

- `app/workspace.tsx`: all page views, dialogs, navigation, interactions and forms.
- `app/globals.css`: exact styles, responsive rules, fonts and animations.
- `app/lib/domain.ts`: validation, calculations, date and timer logic.
- `app/api/data/route.ts`: data reads/writes and authorization.
- `app/chatgpt-auth.ts`: Sites identity helper and sign-in routes.
- `db/schema.ts`, `drizzle/`: actual records schema and migrations.
- `worker/index.ts`, `vite.config.ts`, `scripts/`: Worker runtime and build integration.
- `components/`, `hooks/`, `lib/`: included UI and infrastructure helpers.

Official D1 local-development reference: https://developers.cloudflare.com/d1/best-practices/local-development/
