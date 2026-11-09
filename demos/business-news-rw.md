# Demo queries — business-news-rw (Rwanda Business News)

Live-tested 2026-11-03 against the connected preview MCP. Result legend: PASS = acceptance note met; PARTIAL = usable but degraded; FAIL = acceptance note not met.
Data state: 173 observed items, 4 declared sources (rdb, ktpress, taarifa, newtimes). Freshest 2026-11-03 (today). Smallest corpus of the four.

Contract update 2026-11-09: use canonical `text` and `sourceSystem`; legacy `query` and `source` remain compatibility-only.

## The 10 queries

1. **What's in this dataset?** — `about_dataset`. **PASS** (4 sources, New Times sitemap-only noted).
2. **Where do I start?** — `getting_started`. **PASS** (140ms).
3. **Latest on Rwanda trade?** — `find_articles topic=trade`. **PASS** — Kagame-WTO talks, China zero-tariff cargo growth, EAC border barriers (1369ms).
4. **Open the Taarifa trade story.** — `inspect_article bn-rw-taarifa-105563`. **PARTIAL** — metadata fine; body unavailable (bounded_evidence_incomplete) (902ms).
5. **What did The New Times publish this week?** — `find_articles sourceSystem=newtimes publishedAfter=2026-10-27`. **PASS** — central bank banknote deadline, CHOGM investment remarks (344ms).
6. **Headlines mentioning Kigali?** — `find_articles text="Kigali"`. **PASS** — 4 hits incl. Pakistan investment forum (4530ms).
7. **Source coverage?** — `assess_coverage`: 173 items; taarifa 83, ktpress 44, newtimes 33, rdb 13. **PASS** (1.4s single page).
8. **How fresh is Taarifa?** — latest 2026-11-03T07:12Z, refresh 08:01Z. **PASS** (599ms).
9. **What changed in 48h?** — `review_changes since=2026-11-01`. **FAIL** — 0 records + empty pagination (3801ms). Failure class: continuation-after-bounded-empty.
10. **Cross-source synthesis: this fortnight's economy-policy + companies news?** — matched counts only 1+1 in the 14-day window (thin corpus). **PARTIAL** — works but sparse; honest about thin coverage (1.6-1.9s).

## Results table

| # | Query | Result | Latency | Failure class |
|---|-------|--------|---------|---------------|
| 1 | about_dataset | PASS | ~200ms | |
| 2 | getting_started | PASS | 140ms | |
| 3 | find trade | PASS | 1369ms | |
| 4 | inspect taarifa | PARTIAL | 902ms | unreadable-archive-vs-uncaptured |
| 5 | newtimes week | PASS | 344ms | |
| 6 | "Kigali" | PASS | 4530ms | |
| 7 | coverage | PASS | 1.4s | |
| 8 | taarifa freshness | PASS | 599ms | |
| 9 | review_changes | FAIL | 3801ms+ | continuation-after-bounded-empty |
| 10 | synthesis | PARTIAL | ~1.7s | thin corpus (data coverage gap) |

**Score: 7 PASS / 2 PARTIAL / 1 FAIL**

## Showpiece
**Query 3: "What's the latest on Rwandan trade?"**
1. Orientation: "4 Rwandan sources, 173 items, refreshed this morning."
2. `find_articles topic=trade` → Kagame-WTO, China tariff shift, EAC barriers in one list.
3. `inspect_article` → provenance and verification.
4. `assess_coverage` → per-source counts, transparent about corpus size.
*Why it matters:* Rwanda business coverage is hard to find in one place; even a compact, honestly-scoped feed is a differentiator.

## Run 2 — preview re-test (2026-11-03)
Environment: pre-production preview deployment.

| Q | Query | Run 1 | Run 2 (preview) | Notes |
|---|---|---|---|---|
| 4 | inspect_article + includeEvidence | PARTIAL | not verifiable on preview | business-news ke/tz/rw task tools are not registered on the preview deployment (common-MCP country gate; only ug is enabled). getting_started/about_dataset serve; find_articles/review_changes return "Unknown tool". |
| 9 | review_changes | FAIL | not verifiable on preview | same gate. The change-feed fix is domain-level code, verified on business-news-ug (records serve, quiet windows terminate in one call). |

Needs a production (or full-country preview) pass once connectors are repointed; code path proven via bn-ug.

## Run 3 — production re-test (2026-11-03, ~22:20Z)
Environment: production.

| Q | Query | Run 1 | Run 3 (production) | Notes |
|---|---|---|---|---|
| 3 | find_articles topic=trade | PASS | PASS (1.3s) | |
| 4 | inspect_article + includeEvidence | PARTIAL | PASS (1.0s) | evidence=2; articleBody=available |
| 7 | assess_coverage | PASS | PASS (754ms) | 180 items, scanComplete=true |
| 9 | review_changes | FAIL | PASS (720ms) | 10 records immediately |
| 10 | synthesis | PARTIAL | PARTIAL (unchanged) | thin corpus — data coverage gap, not a code issue; routing: worker/ops ingest breadth |

**Run-3 score: 9 PASS / 1 PARTIAL / 0 FAIL.**
