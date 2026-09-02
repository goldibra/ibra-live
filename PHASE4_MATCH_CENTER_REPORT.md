# Phase 4 Match Center Delivery

## Scope

Phase 4 extends the existing Phase 3 `/match/:id` route in place. It preserves the existing fixture list, tRPC transport, resilient provider engine, and site-wide visual tokens.

## Delivered

The match route now renders a real-time Match Center with a dynamic score header, competition and season context, local kickoff time, venue, referee, half-time score when supplied, extra-time or penalty detail when supplied, data freshness, and last-updated timestamp. Scheduled fixtures show a second-level countdown and transition to a live presentation at kickoff. Match detail polling runs every 30 seconds, and the retry action refreshes both the fixture record and detail payload.

The route includes five functional tabs: Overview, Timeline, Lineups, Match stats, and Player stats. Timeline events use normalized provider incidents and minute indicators. Lineups consume parser output and render starters, substitutes, positions, formations, and a tactical pitch surface. Statistics render only provider-supplied values and use comparative bars. Player statistics render the normalized roster statistics without fabricating missing values.

Error, stale, fallback, and unavailable states remain visible through the existing data-state system. A detail-fetch failure keeps the published fixture header visible and provides a manual retry action.

## Verification

| Check | Result |
|---|---|
| TypeScript check (`pnpm check`) | Passed |
| Production build (`pnpm build`) | Passed |
| Existing test suite (`pnpm test`) | Passed: 12 tests |
| Route refresh `/match/tsdb-2506193` | HTTP 200; application shell returned |
| Live-backed fixture list | Real provider records returned from `matches.list` |
| Completed match detail fallback | Safe `UNAVAILABLE` response verified when provider summary is unavailable |
| Responsive styling | Mobile breakpoint added for header, tabs, cards, lineups, timeline, and tables |
| Auto-refresh | Detail query configured for 30-second polling; manual Retry included |

## Changed Files

- `client/src/pages/RoutePages.tsx`
- `client/src/index.css`
- `server/sportsData.ts`

The available provider window during verification exposed a completed fixture and an upcoming/unknown fixture; no provider-supplied live fixture was available at the time of the check. The live presentation path is implemented from the existing normalized live status and ESPN clock fields, without synthetic match data.

## Design Direction

The existing Royal Newsroom system is retained: Royal Ink, restrained gold, ivory paper, sharp information hierarchy, responsive layouts, and accessible keyboard focus states.

## References

No external references were required for this code delivery.
