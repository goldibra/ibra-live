# Phase 7 Browser Verification Findings

## Home route at `http://localhost:3000/`

The local application rendered successfully after the lazy route module loaded. The browser document title changed from the static shell title to `Real Madrid Live Match Center · IBRA-LIVE-RMA`, confirming client-side metadata injection. The home route rendered the header, navigation, timezone control, theme toggle, alerts button, match content, install prompt, and footer without visible runtime errors. The route returned published fixtures and match links from the existing data layer.

The browser exposed keyboard-reachable links and controls, including the theme toggle and alert button with accessible hints. The home route used the page frame’s default metadata fallback, while the match, team, and player routes are expected to use their entity-specific metadata and JSON-LD.

## Metadata and fixtures deep link

The live head inspection confirmed a canonical URL, description, Open Graph title, Twitter card type, and the route-specific title `Real Madrid Fixtures & Results · IBRA-LIVE-RMA`. The `/matches` deep link rendered successfully with all 47 fixture records, search input, filter buttons, labeled competition/month selects, and navigation controls exposed to the browser.

## Match deep link

The `/match/espn-401882912` deep link initially displayed the loading shell, then hydrated successfully after the data request settled. The document title updated to `Espanyol vs Real Madrid · IBRA-LIVE-RMA`. The match page exposed five keyboard-reachable tabs with `role="tab"`, a retry button, and the match detail content. The final hydrated page showed the match score, venue, incidents, and context sections without a runtime error.

## Match JSON-LD and tabs

The live DOM contained valid `SportsEvent` JSON-LD with event name, ISO start date, completed-event status, venue, home team, away team, and sport. The canonical URL matched the deep link. The active match tab had `aria-selected="true"`, `tabIndex=0`, and controlled `match-panel-overview`; inactive tabs had `tabIndex=-1` and distinct controlled panel IDs. The rendered panel referenced its active tab through `aria-labelledby`.

## Dark theme

The theme toggle switched the match route into dark mode without a runtime error. The browser reported `--paper: #101827`, `--ink: #f3f0e8`, `--muted: #9aa8bc`, and `--gold: #d8b96a`, with body colors/background matching the dark token set. The live page remained readable and all match controls remained reachable.

## Automated viewport verification

Using the active Chromium session with real device metrics, the `/matches` deep link was checked at 1366×768 desktop, 820×1180 tablet, and 390×844 mobile sizes. Each viewport rendered the route-specific title `Real Madrid Fixtures & Results · IBRA-LIVE-RMA`, reported no horizontal overflow, and exposed 26 keyboard-reachable controls with 11 navigation links. The metrics override was cleared after testing.

The same browser session also confirmed that the document starts in RTL (`lang="ar"`, `dir="rtl"`) and can be switched to LTR without a runtime exception. The development server does not register the production-only service worker, so PWA caching was statically validated separately with syntax and policy checks.

## Compiled production server

The compiled server rendered `/matches` successfully with the route-specific title, metadata-backed page shell, accessible search/filter controls, and hydrated fixture data. The initial production navigation briefly displayed the existing loading state and then hydrated to the complete fixture list without a runtime error.

## Production PWA note

The compiled server started on port 3001 because the development process still occupied port 3000. The browser was moved to `http://localhost:3001/matches`, where the compiled bundle rendered correctly. The service-worker asset itself was confirmed to be served with HTTP 200 from the production server and passed `node --check`; the browser registration probe was inconclusive because the prior browser context was still associated with the port-3000 origin and did not control the newly opened port-3001 origin. The service-worker source was also reviewed to confirm precaching of the shell/manifest/offline assets, network-first navigation fallback, and network-only handling for `/api/` live data.

## Production PWA verification completed

After switching to the actual compiled server origin at `http://localhost:3001/`, the browser reported an activated service worker at `/sw.js` controlling the page. The expected shell and runtime caches were present: `ibra-live-rma-phase7-v1-shell` and `ibra-live-rma-phase7-v1-runtime`. This completes the production PWA caching check.

## Team deep link

The compiled production server rendered `/team/real-madrid` successfully after data hydration. The title updated to `Real Madrid team profile · IBRA-LIVE-RMA`, the team profile displayed published fixture metrics and linked match rows, and the route remained free of visible runtime errors. The route uses `SportsTeam` JSON-LD from the shared entity metadata implementation.

## Player deep link

The compiled production server rendered `/player/vinicius-junior` successfully, including its unavailable-record fallback. The title updated to `vinicius-junior player profile · IBRA-LIVE-RMA`, and the page displayed the player profile structure without runtime errors. The route uses the required `Person` JSON-LD payload with optional team affiliation and image fields when published data is available.
