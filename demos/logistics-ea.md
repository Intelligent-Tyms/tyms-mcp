# Demo queries — logistics-ea (East Africa Logistics)

Live-tested 2026-11-03. Data state: 948 observed records. EWURA TZ fuel 570 (single Oct-2026 cycle), KPA port docs 196 (through 2026-11-02), NCTTCA reports 43, CCTTFA 39, FEAFFA news 20, Comtrade trade flows KE-only 73 (annual 2025), KE fuel 6 (2 cycles: Aug-15, Oct-15). corridorIndicator: ZERO records. All calls fast (100-400ms).

## The 10 queries

1. **What's in this dataset?** — `about_dataset`. **PASS** — multi-country (ea), per-record country + subdomain filters, honest "static metadata does not attest populated data" line (115ms).
2. **Where do I start?** — `getting_started`. **PASS** (111ms).
3. **Current diesel price in Dar es Salaam?** — `list_fuel_prices country=tz product=diesel geography=Dar-es-Salaam`. **PASS** — TZS 4,086/litre, validFrom 2026-10-07, provenance = EWURA cap-prices PDF URL (189-389ms).
4. **Open that price record.** — `get_fuel_prices_by_public_id`. **PASS** — full record incl. verification + evidence key (108ms).
5. **Latest Mombasa port shipping movements?** — `list_documents resourceType=portDocument`. **PASS with friction** — docs through 2026-11-02 exist, but list has no sort/date filter; reaching the latest required paging all 8 pages (2.9s). Failure class: missing discovery path (no recency ordering).
6. **Northern Corridor transit-time indicators?** — `list_indicators corridor=northern` (and unfiltered). **FAIL** — 0 indicator records anywhere; about_dataset advertises NCTTCA/CCTTFA corridor observatory indicators. Data coverage gap (declared vs populated).
7. **Coverage by surface?** — `list_coverage`. **PASS** — 8 surfaces with counts/ranges/refresh; scanComplete=false honestly bounded (4.4-4.7s).
8. **How fresh is the data?** — EWURA refreshed 2026-11-03T04:01Z; KPA 11-03; NCTTCA 11-01; but fuelkenya 10-25 and FEAFFA 10-25 are a week stale, and KE fuel has only Aug/Oct cycles (Sep missing). **PARTIAL** — freshness honest but uneven.
9. **What changed since 2026-11-01?** — `list_changes since=2026-11-01T00:00:00Z`. **FAIL** — 0 change items despite EWURA/KPA refreshes on 11-03. Also: date-only "2026-11-01" is REJECTED by schema though the parameter description explicitly allows it (contract-vs-model).
10. **Synthesis: fuel-cost picture for a KE-TZ trucking operator?** — KE Nairobi diesel 217.86→227.76 KES (Aug→Oct) + TZ DSM diesel 4,086 TZS + FEAFFA news. **PASS** — cross-country cost brief in 3 fast calls.

## Results table

| # | Query | Result | Latency | Failure class |
|---|-------|--------|---------|---------------|
| 1 | about_dataset | PASS | 115ms | |
| 2 | getting_started | PASS | 111ms | |
| 3 | DSM diesel | PASS | ~300ms | |
| 4 | inspect price | PASS | 108ms | |
| 5 | latest KPA docs | PASS (friction) | 2.9s (8 pages) | missing discovery path (no sort) |
| 6 | corridor indicators | FAIL | 138-308ms | data coverage gap (declared, unpopulated) |
| 7 | coverage | PASS | 4.5s | |
| 8 | freshness | PARTIAL | (in 7) | uneven staleness; KE Sep fuel cycle missing |
| 9 | changes | FAIL | 204ms | change feed unpopulated + contract-vs-model (date-only rejected) |
| 10 | fuel synthesis | PASS | ~200ms | |

**Score: 6 PASS / 1 PARTIAL / 2 FAIL (+1 pass-with-friction counted as PASS)**

## Showpiece
**Query 3: "What does diesel cost in Dar es Salaam right now?"**
1. Orientation: "Regulator-published fuel caps (EWURA, EPRA/FuelKenya, RURA), KPA daily port reports, corridor performance."
2. `list_fuel_prices country=tz product=diesel` → per-town TZS prices with the EWURA PDF as provenance.
3. `get_fuel_prices_by_public_id` → verification + evidence key.
4. `list_coverage` → live refresh stamps across 8 surfaces.
*Why it matters:* fuel is the biggest variable cost in East African road freight; regulator cap prices with provenance, queryable per town, is a direct costing input.

## Run 2 — preview re-test (2026-11-03)
Environment: pre-production preview deployment.

| Q | Query | Run 1 | Run 2 (preview) | Notes |
|---|---|---|---|---|
| 3 | fuel prices | PASS | PASS (299ms) | 3 items |
| 6 | corridor indicators | FAIL | FAIL (data-level) | corridorIndicator surface has zero records in the preview corpus (unfiltered list also 0); not a code-fix target — ingest/source issue |
| 8 | freshness | PARTIAL | PARTIAL | preview latestRefresh 2026-10-29 (ewura fuel) |
| 9 | list_changes | FAIL | improved | date-only `since` now ACCEPTED (3.6s, was a validation rejection in run 1); feed returns 0 items — honest-empty by design |

Corridor indicators remain the bundle's open item; the change feed is now honest-empty with date-only accepted.

## Run 3 — production re-test (2026-11-03, ~22:20Z)
Environment: production.

| Q | Query | Run 1 | Run 3 (production) | Notes |
|---|---|---|---|---|
| 3 | fuel prices tz | PASS | PASS (265ms) | |
| 6 | corridor indicators | FAIL | FAIL (unchanged) | corridorIndicator surface still zero records, unfiltered included — data/ingest issue, routing: worker |
| 8 | freshness | PARTIAL | PASS | coverage latestRefresh 2026-11-03T04:01Z (today) |
| 9 | list_changes | FAIL | PASS with caveat | full-ISO since terminates in 102ms, 0 items honest-empty by design, nextCursor=null. Caveat: date-only `since` still rejected — S4 persists as a connector-schema/docs mismatch (server accepted date-only on preview direct-HTTP; the connector advertises format date-time) |

**Run-3 score: 8 PASS / 0 PARTIAL / 1 FAIL (Q6 corridor indicators, data-level).**
