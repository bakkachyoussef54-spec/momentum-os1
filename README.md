# Momentum OS

A private productivity workspace built with React, TypeScript, Vinext and Cloudflare D1. The hosted deployment uses ChatGPT sign-in and isolates records by the stable, server-injected user ID. AI Coach is deliberately disconnected, as requested.

## Implemented
- Dashboard with actual saved data, empty states, priority tasks, goals, execution chart, focus overview, habits and deadlines.
- Goals: CRUD, manual numerical progress, milestones, archive/restore, linked tasks, required daily pace.
- Tasks: CRUD, status, scheduling, due dates, priorities, top three, one-level subtasks, recurrence, today/week/all views and filters.
- Twelve-week cycles, up to three linked goals, weekly execution cards and editable reviews.
- Focus sessions: timestamp-based countdown/stopwatch, pause/resume/save, persistent active state, unique active session per user, saved session history.
- Habits: scheduled weekdays, date-based check-ins, target completion, current/best streaks and consistency heatmap.
- Calendar, execution/focus/habit analytics, independent outcomes, manual multi-currency income journal.
- Profile preferences, validated IANA timezone, dark/light themes, week-start setting, dashboard section order, JSON export, delete workspace records, ChatGPT sign-out.
- Guided first-use card and starter goal templates. No performance data is seeded.

## Architecture differences from the brief
This hosting runtime uses Vinext and Cloudflare D1 instead of Next.js on Vercel and Supabase PostgreSQL. Authentication is owned by Sites/ChatGPT, so separate email registration, verification, and password recovery are not provided. Authorization is performed in every API operation; there is no PostgreSQL RLS. This release uses a versioned, schema-validated document table for domain entities with owner indexes, uniqueness keys, and application-enforced relationships, rather than the proposed fully normalized PostgreSQL schema.

The Site remains owner-private. Do not expose the Worker directly or deploy outside the trusted identity-header gateway without replacing authentication. Client-supplied IDs never select the owner. All API responses are private/no-store. Same-origin mutation checks and bounded request bodies are enforced. Optimistic versions reject stale updates; a unique owner/natural-key index protects duplicate check-ins, reviews, settings and active timers.

## Calculations
- Execution = completed / all scheduled tasks excluding cancelled tasks. Subtasks count as actions. Rescheduling changes the action's date bucket. Reopening removes its completion. Recurring instances count separately and are created on completion, not in advance.
- Goal progress is manually entered except milestone-based goals. Completing a linked task does not add numerical goal progress. Progress bars cap at 100%.
- Habit streaks ignore unscheduled days; an unfinished current day does not break the streak until it has passed. Check-ins use the profile timezone and cannot be future-dated.
- Deep work counts saved non-break sessions; a session is assigned to its recorded start date. Midnight sessions are not split across dates. Countdown sessions cap at the chosen duration, stopwatch at 24 hours.
- Currency totals include only entries in the selected base currency. No exchange-rate conversions are performed. Income target is a single reusable monthly target.

## Setup and commands
Node >=22.13.0. Use the project lockfile and pnpm. In Sites, use the plugin's install-dependencies.mjs helper. Standalone dependency installation: `pnpm install --frozen-lockfile`.

- `node node_modules/typescript/bin/tsc --noEmit`
- `node node_modules/eslint/bin/eslint.js app db tests --no-warn-ignored`
- `node --experimental-strip-types --test tests/domain.test.ts`
- `pnpm run db:generate` after changing db/schema.ts
- `pnpm run build`
- `pnpm run dev` for a local Vinext preview

Sites managed preview: `sites-preview start /workspace/sites/momentum-os`. Preview does not simulate hosted identity. The API integration script uses two disposable synthetic identities against this local preview only; never point it at production.

Apply local migrations after a build:
`node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0000_crazy_wrecker.sql`

Hosted migrations are applied by Sites on deployment. Source is synchronized through the Sites source helper before packaging and private publication. Preserve `.openai/hosting.json`, especially the project ID. No API secrets are required for this release. No AI calls are made.

## Known gaps / not production-complete against the entire master brief
AI is deferred by user choice. Remaining requested work includes historical goal snapshots and projected completion, full normalized storage, drag-and-drop task ordering, per-task actual-time display, advance recurring generation, custom analytics date ranges, monthly financial comparison charts, configurable per-month targets, automatic end-of-cycle summaries, notification delivery, richer onboarding, Arabic/RTL translations, pagination for large datasets, comprehensive browser automation, complete mobile/accessibility audit, server-enforced aggregate rate limiting, monitoring integration, and tested backup restoration. Optional external calendar sync/PWA/CSV import are not implemented.

This is a functional first release, not a zero-bug or fully audited production SaaS claim. Do not rely on the entire master brief as a statement of implemented features.

## Backup, recovery and monitoring
Export JSON regularly through Settings. Preserve source/migration history. Database restoration and import are not yet implemented; a JSON export is readable backup data, not a one-click restore tool. Configure provider logging/monitoring and a tested D1 backup/recovery process before broad production use. Never log secrets or user note content.

## Privacy / terms review placeholders
Data entered here includes goals, tasks, habits, focus records, income and reflections. It is stored in the Site's D1 database under your authenticated identity. No AI provider receives it in this release. Export and workspace-data deletion are available. These notes are not a finished privacy policy or terms of service; review provider retention, regional processing, legal notices and support procedures before sharing beyond the owner.

## Verification performed in this build
Production build, TypeScript and scoped ESLint passed. Ten domain tests passed (execution, recurrence counting, progress, habit schedules/streaks, timestamps, timezones and validation). Desktop dashboard was visually inspected in the browser; preview correctly rejected requests without identity. Authenticated UI workflows and the API integration suite could not be verified because the managed preview is not reachable through the executor loopback and the terminal.local HTTP attempt did not complete. No successful integration result is claimed. Mobile layout is implemented with media queries but was not inspected at a mobile viewport. Production sign-in/data smoke testing remains necessary.
