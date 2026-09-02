# Phase 6 — Production Data, Sync Engine, Cache & Failover

## Delivered

The Phase 5 Real Madrid match-center project now has a production-oriented live-data reliability layer spanning provider requests, normalization, persistence, cache behavior, observability, and the UI.

### Live engine and dynamic refresh

The active match bundle now uses the ESPN public feed as the primary source and TheSportsDB as the fallback and shadow-audit source. Provider requests are validated before normalization. The frontend uses the shared polling policy: live matches refresh every 15 seconds, scheduled matches within 24 hours refresh every 60 seconds, and distant/upcoming or inactive records refresh every five minutes. Match detail polling uses the same adaptive policy, so score, status, events, and stats refresh more aggressively for live fixtures.

### Sync pipeline and conflict handling

The sync workflow is implemented as scheduled sync → provider request → validation → normalization → database persistence → cache replacement. Every provider run writes a detailed sync record with provider, timestamps, status, latency, received records, updated records, and errors. ESPN and TheSportsDB records are matched by competition, normalized home/away teams, and UTC match date. Conflicting fields are recorded in `providerConflicts`, and ESPN wins every conflicting field. Fallback data is never labeled as live or verified; the public data state is `FALLBACK`, `STALE`, or `UNAVAILABLE` as appropriate.

### Resilient caching and failover

The in-memory bundle cache is combined with a persistent `dataCache` table. The cache supports a 30-second fresh TTL, a ten-minute stale-while-revalidate window, process-local request deduplication, persistent restart recovery, and stale-cache fallback when providers fail. The provider path is ESPN with retries and exponential backoff, then TheSportsDB, then persistent/in-memory stale cache, then `UNAVAILABLE`. Provider-scoped circuit breakers open after repeated failures and transition through half-open probes. Transport requests have enforced deadlines, including for transports that fail to honor abort signals.

### Scheduler and callback

The server starts an adaptive in-process sync scheduler by default. Its cadence is driven by the fastest match in the current live bundle. A protected `POST /api/scheduled/live-sync` callback is also available for managed heartbeat/cron execution; set `SYNC_CALLBACK_SECRET` to require the `x-sync-secret` header. Set `ENABLE_LIVE_SCHEDULER=false` for local UI-only runs or environments where a managed scheduler invokes the callback instead.

### Data center / status page

A new `/data-center` route shows freshness, source, last/next sync, provider operational status, latency, circuit state, fallback state, the pipeline stages, and recent provider conflicts. The page is linked from the main navigation. Shared `DataStatus` labels now distinguish Live, Verified, Fallback, Stale cache, Unavailable, and Unknown states.

## Database migration

Apply `drizzle/migrations/0000_phase6_production_data.sql` against the existing MySQL/TiDB database before enabling durable sync telemetry and cache persistence. The TypeScript schema in `drizzle/schema.ts` has been updated to match the migration.

## Verification

The following checks passed after the final changes:

| Check | Result |
|---|---|
| `pnpm lint` | Passed |
| `pnpm check` | Passed |
| `pnpm test` | Passed — 4 files, 22 tests |
| `pnpm build` | Passed — Vite client bundle and server bundle produced |
| Browser `/data-center` smoke test | Passed — page, status cards, pipeline, and conflict sections rendered |
| Browser `/matches` smoke test | Passed — live ESPN fixtures loaded with `Verified` freshness and no runtime errors |
| Reliability simulations | Passed — success, retry, circuit breaker, timeout, deduplication, malformed payload, failover, stale cache, conflict resolution, and adaptive polling |

Browser evidence is retained in `PHASE6_BROWSER_VERIFICATION.md`.
