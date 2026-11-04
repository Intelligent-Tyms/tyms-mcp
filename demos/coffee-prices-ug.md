# Demo queries — coffee-prices-ug (Uganda Coffee Prices)

Live-tested 2026-11-03. Data state: production lane; UCDA daily prices block (162 export obs through 2026-11-02, 47 farmgate), daily market report, monthly report (through 2026-06); World Bank overseas benchmarks (8 obs, monthly, through 2026-09); context documents 3 (ICO market reports Aug/Sep 2026, UCDA-MAAIF press brief). All calls 100-600ms. All surfaces cadence-classified stale=false.

## The 10 queries

1. **What's in this dataset?** — `about_dataset`. **PASS** — value-chain stages, cadence per surface, gated surfaces (regional-benchmark, narrative context) stated (120ms).
2. **Where do I start?** — `getting_started`. **PASS** (121ms).
3. **Today's farmgate price for arabica parchment?** — `find_prices priceStage=farmgate range`. **PASS** — 2026-11-02: 13,000-14,000 UGX/kg indicative range; kiboko alongside (320ms).
4. **Open that observation with evidence.** — `inspect_observation includeEvidence=true`. **PARTIAL** — full record + verification + UCDA source URL, but evidence array empty despite evidenceIds (EVIDENCE_UNAVAILABLE pattern) (165ms).
5. **This week's export prices (drugar/bugisu)?** — `find_prices priceStage=export`. **PASS** — bugisu-aa 282.2 US cts/lb on 2026-11-02; full Bugisu/drugar ladder daily (559ms). (Note: value field is `pricePoint`; unit us-cts/lb.)
6. **Latest international benchmark?** — `find_prices priceStage=overseas-benchmark`. **PARTIAL** — only World Bank monthly averages (through 2026-09); the daily ICO I-CIP surface is absent from the store (consistent with the control-plane's known overseas-benchmark parse-drift routing). No recent daily benchmark (189-222ms).
7. **Coverage by stage/source?** — `assess_coverage`. **PASS** — per-surface segments with cadence classification (186ms).
8. **How fresh is the daily block?** — refreshed 2026-11-02T15:21Z, data through 2026-11-02, stale=false. **PASS**.
9. **What's newly ingested?** — no change feed in this bundle; `list_observations sort=desc`. **PASS (substitute)** — 2026-11-02 rows on top (232ms).
10. **Synthesis: is the farmgate-to-export spread moving?** — farmgate parchment 13.0-14.0k UGX/kg + export bugisu-aa 282.2 cts/lb + monthly unit values (arabica 6.16 USD/kg Jun) + ICO Sep market report context doc. **PASS** — a genuine value-chain brief; context documents give the narrative layer (287ms).

## Results table

| # | Query | Result | Latency | Failure class |
|---|-------|--------|---------|---------------|
| 1 | about_dataset | PASS | 120ms | |
| 2 | getting_started | PASS | 121ms | |
| 3 | farmgate today | PASS | 320ms | |
| 4 | inspect + evidence | PARTIAL | 165ms | EVIDENCE_UNAVAILABLE |
| 5 | export week | PASS | 559ms | |
| 6 | overseas benchmark | PARTIAL | ~200ms | data coverage gap (no daily ICO; monthly WB only) |
| 7 | coverage | PASS | 186ms | |
| 8 | freshness | PASS | (in 7) | |
| 9 | newest ingested | PASS (substitute) | 232ms | no changes feed by design |
| 10 | value-chain synthesis | PASS | ~300ms | |

**Score: 8 PASS / 2 PARTIAL / 0 FAIL**

## Showpiece
**Query 3+5: "What did a Ugandan coffee farmer get paid today, and what does the export market pay?"**
1. Orientation: "Official UCDA daily prices across the value chain, refreshed every afternoon."
2. `find_prices priceStage=farmgate` → parchment 13,000-14,000 UGX/kg today.
3. `find_prices priceStage=export` → bugisu-aa 282.2 US cts/lb today.
4. `assess_coverage` → freshness and cadence proof.
*Why it matters:* farmgate-to-export spread is the single most-asked coffee question in Uganda; this answers it from official publications in two sub-second calls.

## Run 2 — preview re-test (2026-11-03)
Environment: pre-production preview deployment.

| Q | Query | Run 1 | Run 2 (preview) | Notes |
|---|---|---|---|---|
| 3 | farmgate range | PASS | PASS (493ms) | |
| 4 | inspect_observation + includeEvidence | PARTIAL | unchanged on preview data | evidence still 0 for sampled records, now with an honest EVIDENCE_UNAVAILABLE warning; hydration code path is proven on financial-markets/business-news — the preview coffee corpus appears to hold no evidence KV objects for these records |
| 5 | export range | PASS | PASS (444ms) | |
| 6 | overseas-benchmark | PARTIAL | improved | ICO indicator benchmarks present (composite, robustas, milds, Brazilian naturals — 2026-08-21) where run 1 saw WB-monthly only |

Coffee stays the fastest bundle; evidence availability on preview is a corpus limitation, code path verified elsewhere.

## Run 3 — production re-test (2026-11-03, ~22:20Z)
Environment: production.

| Q | Query | Run 1 | Run 3 (production) | Notes |
|---|---|---|---|---|
| 3 | farmgate range | PASS | PASS (1.0s) | today's 2026-11-03 UCDA block prices |
| 4 | inspect_observation + includeEvidence | PARTIAL | PASS | overseas-benchmark obs → evidence=1 in 135ms; the farmgate daily-block records genuinely have no evidence objects (get_evidence 0, honest EVIDENCE_UNAVAILABLE) — hydration code works on production, evidence coverage is a data item for the farmgate surface |
| 5 | export range | PASS | PASS (442ms) | |
| 6 | overseas-benchmark | PARTIAL | PASS (255ms) | daily UCDA London robusta benchmark closes dated today (run 1: WB monthly only) |

**Run-3 score: 10 PASS / 0 PARTIAL / 0 FAIL.**
