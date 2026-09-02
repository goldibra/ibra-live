# IBRA-LIVE-RMA — Phase 2 Execution Report

## Scope

Phase 2 was implemented directly on top of the supplied Phase 1 stabilized codebase. Existing Phase 1 sports endpoints and UI-facing contracts were preserved; the new engine is additive and exposed through `matches.engine`.

## Implemented

| Area | Result |
|---|---|
| ESPN provider | Added endpoint adapter methods for matches, competitions, standings, scorers, lineups, stats, and events. Requests propagate the selected season. |
| TheSportsDB fallback | Added a strictly secondary adapter. It is not used for unknown competitions and has no LaLiga defaulting behavior. |
| Resilience | Added timeout control, exponential retry/backoff, concurrent request deduplication, primary-to-fallback execution, in-memory cache recovery, and explicit unavailable state. |
| Validation | Added Zod validation for raw ESPN and TheSportsDB response envelopes. Invalid payloads fail gracefully into fallback/cache/unavailable handling. |
| Normalization | Added unified teams, matches, dataset metadata, and provider-independent competition/season structures. |
| Status mapping | Added `SCHEDULED`, `LIVE`, `HALFTIME`, `EXTRA_TIME`, `PENALTIES`, `FINISHED`, `POSTPONED`, `CANCELLED`, and `UNKNOWN`. |
| Competition detection | Added LaLiga, Champions League, Copa del Rey, Spanish Super Cup, Friendly, and Unknown detection. Unknown competitions remain Unknown. |
| Season engine | Added explicit `YYYY-YYYY` season context with propagated ID, label, and boundaries. |
| Sync service | Added `Fetch → Validate → Normalize → Persist → Invalidate Cache` orchestration and polling intervals for live versus idle data. |
| Database | Added nullable `seasonId` to the existing `matches` table and carried it through match upserts. |
| Frontend API | Added `matches.engine`, with competition and season inputs and metadata-bearing provider-agnostic results. |

## Files added or changed

| File | Change |
|---|---|
| `server/sportsEngine.ts` | New Phase 2 provider, validation, normalization, fallback, cache, and sync engine. |
| `server/sportsEngine.test.ts` | New tests for status mapping, competition/season detection, normalization, malformed payloads, fallback, caching, and deduplication. |
| `server/routers.ts` | Added season-aware `matches.engine` API route. |
| `server/db.ts` | Persisted `seasonId` during match upserts. |
| `drizzle/schema.ts` | Added `matches.seasonId`. |
| `server/sportsData.ts` | Added optional `seasonId` to the existing Phase 1 match record. |

## Verification

| Command | Result |
|---|---|
| `pnpm check` | Passed. |
| `pnpm test -- --reporter=dot` | Passed: 3 test files, 12 tests. |
| `pnpm build` | Passed: Vite client build and bundled server build completed. |

## Operational notes

The new engine uses injectable request functions, which keeps provider parsing and resilience behavior deterministic under tests and makes future API-key or transport changes localized. Metadata is attached to every new dataset, and cache recovery is explicitly marked `source: CACHE`, `freshness: STALE`, and `status: STALE`; stale data is never labeled as live.

The database schema change requires the normal project migration/push step in an environment with `DATABASE_URL` before production use.
