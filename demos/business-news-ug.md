# Demo queries — business-news-ug (Uganda Business News)

Live-tested 2026-11-03 against the connected preview MCP. Result legend: PASS = acceptance note met; PARTIAL = usable but degraded; FAIL = acceptance note not met.
Data state at test time: ~2,141 observed items across the 58 declared sources in the visible window; freshest items 2026-11-03. Evidence modes: mostly full-record; ceoea/pmldaily reference-only; kilimo link-only.

Contract update 2026-11-09: use canonical `text` and `sourceSystem`; legacy `query` and `source` remain compatibility-only.

## The 10 queries

1. **What's in this dataset?** (cold-start) — trajectory: `about_dataset`. Accept: lists sources, 6-hour cadence target, licence posture. **PASS** (122-200ms doc call).
2. **Where do I start?** (cold-start) — `getting_started`. Accept: points at about_dataset + task tools. **PASS** (122ms).
3. **What's the latest on Uganda coffee?** (core find) — `find_articles topic=coffee limit=5`. Accept: >=3 recent headlines with publicIds. **PASS** — 5 hits (observer, foodbusinessmea, ceoea, softpower, nilepost) (336ms).
4. **Open the Observer coffee story.** (core inspect) — `inspect_article bn-ug-observer-73921`. Accept: full record, verification, rights-permitted body. **PARTIAL** — record + SOURCE_ATTESTED verification fine, but `articleBody.status=withheld (source_exposure_posture)` and `includeEvidence=true` returns 0 evidence despite evidenceIds present. Body never readable (1297ms).
5. **What did Nile Post publish this week?** (core find) — `find_articles sourceSystem=nilepost publishedAfter=2026-10-27`. Accept: dated headlines from nilepost only. **PASS** — 5 fresh items (287ms).
6. **Any headlines mentioning Bank of Uganda?** (core find, substring) — `find_articles text="Bank of Uganda"`. Accept: matching headlines. **PASS but slow** — 5 hits, 9.5s latency (substring scan) (9453ms).
7. **Which sources are covered and how much from each?** (coverage) — `assess_coverage` paginated. Accept: per-source counts + freshness + evidence modes. **PASS** — 5 pages, 128 windowed segments, ~2141 items (14s for full pagination).
8. **How fresh is The Observer?** (freshness) — `assess_coverage sourceSystem=observer`. Accept: latestPublishedAt within days. **PASS** — 2026-11-01T13:23Z, refresh 15:00Z (411ms).
9. **What changed in the last 48 hours?** (what's-new) — `review_changes since=2026-11-01`. Accept: change records. **FAIL** — 0 records; pagination emits 40+ consecutive empty pages (~4s each, 168s abandoned) despite items published in-window. Failure class: continuation-after-bounded-empty (change feed unpopulated).
10. **What are this week's top business themes?** (synthesis) — `find_articles topic=energy` + `topic=banking` windowed, cite publicIds. Accept: cross-source synthesis with citations. **PASS** — Umeme court ruling, solar investment push (energy); dfcu 16.9% home loans across 3 sources (banking) (736ms).

## Results table

| # | Query | Result | Latency | Failure class |
|---|-------|--------|---------|---------------|
| 1 | about_dataset | PASS | ~200ms | |
| 2 | getting_started | PASS | 122ms | |
| 3 | find coffee | PASS | 336ms | |
| 4 | inspect observer article | PARTIAL | 1297ms | rights-withheld-vs-absent + EVIDENCE_UNAVAILABLE |
| 5 | nilepost week | PASS | 287ms | |
| 6 | "Bank of Uganda" | PASS (slow) | 9453ms | latency: substring scan |
| 7 | coverage paginated | PASS | 14s (5 pages) | |
| 8 | observer freshness | PASS | 411ms | |
| 9 | review_changes 48h | FAIL | >168s (abandoned) | continuation-after-bounded-empty |
| 10 | themes synthesis | PASS | 736ms | |

**Score: 8 PASS / 1 PARTIAL / 1 FAIL**

## Showpiece (60-second demo)
**Query 3+4: "What's the latest on Uganda coffee?"**
1. Orientation line: "58 Ugandan sources, refreshed within 6 hours on business days, ~2,100 items visible now."
2. `find_articles topic=coffee` → instant 5 headlines (Mbale-China coffee markets, AU export strategy, Chinese buyers cutting middlemen).
3. `inspect_article` on the Observer piece → full metadata, SOURCE_ATTESTED verification, canonical URL, evidence IDs.
4. `assess_coverage sourceSystem=observer` → 48 items, latest 2026-11-01.
*Why it matters for a buyer:* one call turns "monitor the Ugandan coffee beat" from a scraping project into a query — with per-publisher attribution and rights posture attached.

## Run 2 — preview re-test (2026-11-03)
Environment: pre-production preview deployment.

| Q | Query | Run 1 | Run 2 (preview) | Notes |
|---|---|---|---|---|
| 3 | find_articles topic=coffee | PASS | PASS (1.4s) | 3 fresh-looking records from Sept corpus |
| 4 | inspect_article + includeEvidence | PARTIAL | PASS (165ms) | evidence now resolves (1 record); preview schema has no includeBody argument |
| 7 | assess_coverage | PASS | PASS (78ms) | 215 observed items on preview |
| 9 | review_changes | FAIL | PASS | since=2026-08-01 → 10 change records in 2.4s with working pagination; quiet window (since=2026-11-01) terminates in 1 call / 190ms, complete=true, honest EMPTY_RESULT |

Run-1 FAIL/PARTIAL both cleared on the fixed build. Equivalent run-2 score: 10 PASS / 0 PARTIAL / 0 FAIL.

## Run 3 — production re-test (2026-11-03, ~22:20Z)
Environment: production.

| Q | Query | Run 1 | Run 3 (production) | Notes |
|---|---|---|---|---|
| 3 | find_articles topic=coffee | PASS | PASS (449ms) | |
| 4 | inspect_article + includeEvidence | PARTIAL | PASS (316ms) | evidence=2; articleBody=withheld is the honest rights posture for that source |
| 7 | assess_coverage | PASS | PASS (2.2s) | |
| 9 | review_changes | FAIL | PASS (1.6s) | 10 records immediately; deploy session walked today's feed to termination (11 pages, 270 records, complete=true) |

**Run-3 score: 10 PASS / 0 PARTIAL / 0 FAIL.**
