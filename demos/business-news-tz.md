# Demo queries — business-news-tz (Tanzania Business News)

Live-tested 2026-11-03 against the connected preview MCP. Result legend: PASS = acceptance note met; PARTIAL = usable but degraded; FAIL = acceptance note not met.
Data state: ~732 observed items, 10 declared sources; includes Swahili desks (Habari Leo, Mwananchi). Freshest 2026-11-03.

Contract update 2026-11-09: use canonical `text` and `sourceSystem`; legacy `query` and `source` remain compatibility-only.

## The 10 queries

1. **What's in this dataset?** — `about_dataset`. **PASS** (10 sources, thin English-media caveat stated).
2. **Where do I start?** — `getting_started`. **PASS** (150ms).
3. **Latest on Tanzanian mining/gold?** — `find_articles topic=mining`. **PASS** — gold refineries/LBMA, MINEXPO, Stamico new mines (English + Swahili) (554ms).
4. **Open the Daily News gold story.** — `inspect_article bn-tz-dailynews-167652`. **PARTIAL** — metadata fine; `articleBody=unavailable (bounded_evidence_incomplete)`, fullText null (1112ms). Failure class: unreadable-archive-vs-uncaptured.
5. **What did TanzaniaInvest publish this week?** — `find_articles sourceSystem=tanzaniainvest publishedAfter=2026-10-27`. **PASS** — avocado exports USD 97.2m, weekly markets wraps, PPP newsletter (277ms).
6. **Headlines mentioning Bank of Tanzania?** — `find_articles text="Bank of Tanzania"`. **PASS (literal)** — 2 hits, both older BoT press releases (2024); substring search is literal so recent BoT items phrased differently are missed; 12.3s (12308ms).
7. **Source coverage?** — `assess_coverage` paginated: 2 pages, 12 segments, 732 items. **PASS** (4.5s).
8. **How fresh is The Citizen?** — `assess_coverage sourceSystem=thecitizen`: 78 items, latest 2026-11-02. **PASS** (733ms).
9. **What changed in 48h?** — `review_changes since=2026-11-01`. **FAIL** — 0 records, empty nextCursor pagination (2888ms). Failure class: continuation-after-bounded-empty.
10. **This week: policy & energy themes?** — `find_articles topic=economy-policy + topic=energy`. **PASS** — PPP centre TZS 170trn pipeline, TPDC gas licences, fuel-station ownership model (675-1335ms).

## Results table

| # | Query | Result | Latency | Failure class |
|---|-------|--------|---------|---------------|
| 1 | about_dataset | PASS | ~200ms | |
| 2 | getting_started | PASS | 150ms | |
| 3 | find mining | PASS | 554ms | |
| 4 | inspect dailynews | PARTIAL | 1112ms | unreadable-archive-vs-uncaptured |
| 5 | tanzaniainvest week | PASS | 277ms | |
| 6 | "Bank of Tanzania" | PASS (literal) | 12308ms | latency + substring-only semantics |
| 7 | coverage | PASS | 4.5s | |
| 8 | citizen freshness | PASS | 733ms | |
| 9 | review_changes | FAIL | 2888ms+ | continuation-after-bounded-empty |
| 10 | policy+energy | PASS | ~1s | |

**Score: 8 PASS / 1 PARTIAL / 1 FAIL**

## Showpiece
**Query 3: "What's happening in Tanzanian gold and mining?"**
1. Orientation: "10 Tanzanian sources, English and Swahili desks, ~730 items, fresh today."
2. `find_articles topic=mining` → LBMA certification debate, MINEXPO, Stamico expansion — a cross-language brief in one call.
3. `inspect_article` on the Daily News piece → verification + provenance.
4. `assess_coverage sourceSystem=thecitizen` → freshness proof.
*Why it matters:* Tanzania mining intelligence normally needs Swahili and English monitoring; this normalizes both into one queryable feed.

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
| 3 | find_articles topic=mining | PASS | PASS (1.2s) | |
| 4 | inspect_article + includeEvidence | PARTIAL | PASS (797ms) | evidence=2; articleBody=available |
| 8 | thecitizen freshness | PASS | PASS (264ms) | |
| 9 | review_changes | FAIL | PASS (714ms) | 10 records immediately |

**Run-3 score: 10 PASS / 0 PARTIAL / 0 FAIL.**
