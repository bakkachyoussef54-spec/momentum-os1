# Momentum OS — original source export

This ZIP contains the actual source of the existing Momentum OS website. It is an export, not a reconstruction, redesign or independently rewritten app.

- Website: https://momentum-os-youssef.youngdog4.chatgpt.site
- Source commit: `f415e2c0bb21d3e7a91f69d8e9887820287bb906`
- Published version checked: 1
- Export date: 2026-10-27
- All Git-tracked source files are preserved byte-for-byte, including original configuration and README.
- The existing `.env.example` and generated `next-env.d.ts` are included as supplementary project files.
- `EXPORT-*.md` and `EXPORT-MANIFEST.json` are new export documentation only.
- The live website, database, access settings and deployment were not modified.

## Read these first

1. `EXPORT-LOCAL-DEVELOPMENT.md`: dependencies, local database, development server, local sign-in and tests.
2. `EXPORT-VERCEL-MIGRATION.md`: the specific infrastructure changes required before Vercel hosting.
3. `README.md`: the original implementation notes and known gaps.
4. `EXPORT-MANIFEST.json`: complete original-file inventory and SHA-256 checksums.

## Important compatibility limit

**The untouched original is not ready to deploy directly to Vercel.** Its server code imports `cloudflare:workers`, its database uses a D1 binding, and authentication relies on the ChatGPT Sites gateway. Setting environment variables or selecting the Next.js preset alone does not replace those services.

The original UI, components, CSS, domain logic and API behavior are retained. A separate infrastructure migration is necessary to make them independently deployable. No such rewrite is disguised as original source in this archive.

## Included and excluded

Included: frontend and backend, all existing components, schema, SQL migrations and Drizzle metadata, scripts, dependency manifest and exact lockfile, configuration (including `.openai/hosting.json`), public favicon, tests, examples and bundled licenses.

Excluded: `node_modules`, generated builds/caches, `.git` history and credentials, local runtime state, logs, actual secret files and local database files. Dependencies are restored from the lockfile. The retained project ID is an identifier, not a credential; do not use it to publish changes to the existing Site accidentally.

Your hosted goals, tasks, habits and income records are **not** source files and are not included. Use the live application's Settings → Export all data (JSON) for a separate data backup. Its import workflow is not implemented; a migration must remap records to the new authenticated account.

Google Fonts are referenced remotely by the original CSS (DM Sans and Manrope). This preserves the existing font loading behavior; it does not bundle font binaries. Lucide icons are supplied by the dependency lockfile. Existing animations and all authored styling are in `app/globals.css`.

## Verification

The ZIP was checked for CRC errors and every original file was compared byte-for-byte with the available source. All tracked files match the published commit above. The export contains no dependency or infrastructure upgrades. No claim is made that a Vercel build has passed: the migration is still required. Prior application testing and limitations remain documented in the original README.
