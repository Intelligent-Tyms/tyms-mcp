# Demo queries — mining-ug (Uganda Mining)

Live-tested 2026-11-03. Data state: 767 records — cadastre register 308 licences (refresh 2026-06-13, ~5 months stale vs declared 24h delta detection), gazette grant events 459 (refresh 2026-09-25, ~5 weeks stale vs 48h claim). Dealers register gated-planned, 0 published. Status: preview-only.

## The 10 queries

1. **What's in this dataset?** — `about_dataset`. **PASS** — register semantics (re-observed entity + append-only events), gating and cadence stated (144ms).
2. **Where do I start?** — `getting_started`. **PASS** (170ms).
3. **Show active mineral licences.** — `list_licences limit=5`. **PASS** — AML artisanal + legacy EL exploration licences with status (240ms).
4. **Open licence AML00644.** — `get_licences_by_public_id`. **PARTIAL** — full attributes + verification, but `sourceAttribution.sourceUrls=[]`: cadastre records carry no provenance URL (unlike other bundles) (308ms). Failure class: missing provenance.
5. **Recent gazette grant notices?** — `list_grant_events limit=5`. **PASS with friction** — events returned but oldest-first (2023-10-20) with no sort/date filter; reaching 2026 grants needs paging (253ms).
6. **Who holds a dealer's licence?** — `list_licences sourceSurface=dealers-register`. **PARTIAL (honest empty)** — 0 items; coverage correctly reports the surface as gated-planned/unpublished, but discovery returns empty page WITH a nextCursor (mild continuation-after-bounded-empty) (21.9s for an empty answer).
7. **Coverage by surface?** — `list_coverage`. **PASS** — surfaces, planned surfaces with gating notes, refresh stamps (6.2s).
8. **How fresh is the register?** — cadastre refresh 2026-06-13 (vs 24h declared), gazette 2026-09-25 (vs 48h declared). **FAIL (staleness)** — both surfaces weeks-to-months behind declared cadence. Failure class: stale-source / publish-health.
9. **What changed since September?** — `list_changes since=2026-09-01T00:00:00Z`. **FAIL** — 0 change items despite a gazette refresh on 2026-09-25 (change feed unpopulated) (204ms).
10. **Cross-record: licence EL00308 and its grant history?** — `list_licences licenceCode=EL00308` + `list_grant_events licenceCode=EL00308` + `list_evidence`. **PASS** — active licence + 2023-10-20 grant event + evidence record join cleanly. Slow: 35s + 12s for filtered lookups.

## Results table

| # | Query | Result | Latency | Failure class |
|---|-------|--------|---------|---------------|
| 1 | about_dataset | PASS | 144ms | |
| 2 | getting_started | PASS | 170ms | |
| 3 | list licences | PASS | 240ms | |
| 4 | inspect licence | PARTIAL | 308ms | missing provenance (empty sourceUrls) |
| 5 | grant events | PASS (friction) | 253ms | no sort/date filter |
| 6 | dealers register | PARTIAL | 21.9s | gated surface; empty+cursor page |
| 7 | coverage | PASS | 6.2s | |
| 8 | freshness | FAIL | (in 7) | stale-source (cadastre ~5mo, gazette ~5wk) |
| 9 | changes | FAIL | 204ms | change feed unpopulated |
| 10 | licence+events synthesis | PASS | 35s+12s | latency on filtered queries |

**Score: 6 PASS / 2 PARTIAL / 2 FAIL**

## Showpiece
**Query 10: "Who holds exploration licence EL00308 and when was it granted?"**
1. Orientation: "Uganda's mineral-rights register re-observed from the DGSM cadastre, plus gazette grant notices — 767 records."
2. `list_licences licenceCode=EL00308` → active exploration licence with full attributes.
3. `list_grant_events licenceCode=EL00308` → its statutory gazette grant (GN 1759, 20 Oct 2023).
4. `list_coverage` → provenance posture and refresh state, honestly including staleness.
*Why it matters:* joining the live cadastre to statutory grant notices is a mineral-rights due-diligence primitive that today requires portal scraping and gazette PDF reading. (Demo caveat: cite refresh dates; register is weeks stale.)

## Run 2 — preview re-test (2026-11-03)
Environment: pre-production preview deployment.

| Q | Query | Run 1 | Run 2 (preview) | Notes |
|---|---|---|---|---|
| 3 | list_licences | PASS | PASS (187ms) | |
| 4 | licence sourceUrls | PARTIAL | unchanged | sourceUrls still absent on licence records (not a fix target this deploy) |
| 6 | dealers-register | PARTIAL | unchanged | still 0 items after a 45s paginating scan — remains open |
| 8 | cadastre staleness | FAIL | unchanged (expected) | MN-1 diagnosed not-a-code-bug, routed to ops; preview cadastre latestRefresh 2026-06-13 |
| 9 | list_changes | FAIL | improved | date-only `since` accepted; feed returns 0 items — honest-empty by design |

No regressions; the two data-level items (sourceUrls, dealers register) and ops staleness carry forward.

## Run 3 — production re-test (2026-11-03, ~22:20Z)
Environment: production.

| Q | Query | Run 1 | Run 3 (production) | Notes |
|---|---|---|---|---|
| 3 | list_licences | PASS | PASS (931ms) | |
| 4 | licence sourceUrls | PARTIAL | PARTIAL (unchanged) | sourceUrls still absent — data/adapter item, routing: worker |
| 6 | dealers-register | PARTIAL | PARTIAL (unchanged) | still 0 items after a 25s paginating scan — data + latency, routing: worker/ops |
| 8 | cadastre staleness | FAIL | FAIL (expected) | cadastre latestRefresh 2026-06-13 (~5 months vs 24h declared) — MN-1, ops-routed, not a code bug |
| 9 | list_changes | FAIL | PASS | full-ISO since terminates 138ms, 0 items honest-empty by design, nextCursor=null |

**Run-3 score: 7 PASS / 2 PARTIAL / 1 FAIL.**
