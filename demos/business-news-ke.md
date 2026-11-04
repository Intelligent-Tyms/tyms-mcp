# Demo queries — business-news-ke (Kenya Business News)

Live-tested 2026-11-03 against the connected preview MCP. Result legend: PASS = acceptance note met; PARTIAL = usable but degraded; FAIL = acceptance note not met.
Data state: ~2,042 observed items, 15 declared sources; businessdaily link-only (robots), freshest 2026-11-03. KBC feed includes general (non-business) news items.


## The 10 queries

1. **What's in this dataset?** — `about_dataset`. **PASS** (15 sources, limitations incl. Business Daily robots posture).
2. **Where do I start?** — `getting_started`. **PASS** (201ms).
3. **Latest on Kenyan interest rates / T-bills?** — `find_articles topic=rates`. **PASS** — 5 hits incl. CBK T-bill auction, 91-day T-bill demand (873ms).
4. **Open the Bizna T-bill story.** — `inspect_article bn-ke-bizna-265386 includeBody`. **PARTIAL** — metadata fine but `articleBody.status=unavailable (bounded_evidence_incomplete)`, fullText null on a full-record source (1131ms). Failure class: unreadable-archive-vs-uncaptured.
5. **What did KBC publish this week?** — `find_articles source=kbc publishedAfter=2026-10-27`. **PASS with caveat** — returns items but several are general news (sport, courts), not business: feed-level classification noise (252ms).
6. **Headlines mentioning Central Bank?** — `find_articles query="Central Bank"`. **PASS but slow** — 3 hits, 19.4s (19385ms).
7. **Source coverage and volumes?** — `assess_coverage` paginated. **PASS** — 5 pages, 58 segments, ~2042 items; businessdaily correctly shown link-only (13s).
8. **How fresh is the corpus?** — segments show bizna/capitalbusiness/sokodirectory at 2026-11-03. **PASS**.
9. **What changed in the last 48 hours?** — `review_changes since=2026-11-01`. **FAIL** — 0 records + empty nextCursor pagination (same systemic pattern) (2917ms).
10. **This week in Kenyan banking & capital markets?** — `find_articles topic=banking` + `topic=capital-markets`. **PASS** — CBK vs Mbadi on bank capital rule, record Sh1trn bond bids, NSE teen investors (292-387ms).
Extra: **Business Daily rights check** — `inspect_article` on a businessdaily ID: `articleBody=withheld (source_exposure_posture)`, rightsPosture=headline_link_only. **PASS (contract works as designed)** — the honest link-only path.

## Results table

| # | Query | Result | Latency | Failure class |
|---|-------|--------|---------|---------------|
| 1 | about_dataset | PASS | ~200ms | |
| 2 | getting_started | PASS | 201ms | |
| 3 | find rates | PASS | 873ms | |
| 4 | inspect bizna | PARTIAL | 1131ms | unreadable-archive-vs-uncaptured |
| 5 | kbc week | PASS (noisy) | 252ms | classification noise (not-business items) |
| 6 | "Central Bank" | PASS (slow) | 19385ms | latency |
| 7 | coverage | PASS | 13s | |
| 8 | freshness | PASS | (in 7) | |
| 9 | review_changes | FAIL | 2917ms+ | continuation-after-bounded-empty |
| 10 | banking+capmkts | PASS | ~300ms | |

**Score: 8 PASS / 1 PARTIAL / 1 FAIL** (Q5 passes with a data-quality caveat)

## Showpiece
**Query 10: "What's moving Kenyan banking and capital markets this week?"**
1. Orientation: "15 Kenyan sources including CBK, NSE, Business Daily; ~2,000 items, freshest today."
2. `find_articles topic=capital-markets publishedAfter=<7d>` → record Sh1trn bond demand, NSE rally stories.
3. `inspect_article` on the CBK-vs-Mbadi story → shows the link-only rights contract honestly (withheld, canonical URL).
4. `assess_coverage` → per-source volumes prove breadth.
*Why it matters:* a markets analyst gets a curated cross-source brief in two calls, and the rights posture is explicit per publisher.

## Run 2 — preview re-test (2026-11-03)
Environment: pre-production preview deployment.

| Q | Query | Run 1 | Run 2 (preview) | Notes |
|---|---|---|---|---|
| 4 | inspect_article + includeEvidence | PARTIAL | not verifiable on preview | business-news ke/tz/rw task tools are not registered on the preview deployment (common-MCP country gate; only ug is enabled). getting_started/about_dataset serve; find_articles/review_changes return "Unknown tool". |
| 9 | review_changes | FAIL | not verifiable on preview | same gate. The change-feed fix is domain-level code, verified on business-news-ug (records serve, quiet windows terminate in one call). |

Needs a production (or full-country preview) pass once connectors are repointed; code path proven via bn-ug.

## Run 3 — production re-test (2026-11-03, ~22:20Z)
Environment: production. The run-2 blocker (task tools gated off on preview) does not apply on production: all four countries (ug, rw, ke, tz) are enabled.

| Q | Query | Run 1 | Run 3 (production) | Notes |
|---|---|---|---|---|
| 3 | find_articles topic=rates | PASS | PASS (1.6s) | |
| 4 | inspect_article + includeEvidence | PARTIAL | PASS (1.9s) | evidence=2; articleBody=available (kbc full text) |
| 5 | kbc week sample | PASS | PASS (266ms) | |
| 9 | review_changes | FAIL | PASS (506ms) | 10 records immediately, pagination live |

**Run-3 score: 10 PASS / 0 PARTIAL / 0 FAIL.**
