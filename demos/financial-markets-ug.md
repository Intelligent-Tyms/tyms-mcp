# Demo queries — financial-markets-ug (Uganda Financial Markets)

Live-tested 2026-11-03. Data state: production lane; FX 2,159 obs through 2026-11-03 (today), securities-market 9,672 obs, bonds 131, money-markets 222, macro 586, external-sector 432, equities 27. Sources: BoU + USE.

Contract update 2026-11-09: `find_observations` is canonical; `get_observations` remains compatibility-only.

## Tiered eval set (v2, 2026-11-07)

Skill-A/B eval shape per the internal workflow plan §4.
Tier A = simple lookup, **no skill by design** (control group). Tier B = directed
multi-tool, skill-optional. Tier C = workflow, each mapped to exactly one skill.
The original 10 queries below remain the regression core; IDs in parentheses map
old queries into the tiers. Acceptance = machine-checkable first, judge second.

### Tier A — simple lookup (skill: none)

| ID | Query | Expected path | Acceptance |
|----|-------|---------------|------------|
| FMU-A1 (Q1) | What's in this dataset? | `about_dataset` | 8 surfaces + BoU/USE sources named; verification posture stated |
| FMU-A2 (Q2) | Where do I start? | `getting_started` | Points at find_series → find_observations path |
| FMU-A3 (Q3a) | Find the USD FX series | `find_series text=USD surface=fx` | Series metadata + seriesIds; deduped by seriesId; no market values claimed |
| FMU-A4 (Q4) | Inspect observation \<id\> with evidence | `inspect_observation includeEvidence=true` | Full record; empty evidence despite evidenceIds reported as unavailable, never fabricated |
| FMU-A5 (Q7) | Coverage by surface/source? | `assess_coverage` | Per-surface counts/ranges/refresh quoted as observed-window, not market totals |

### Tier B — directed multi-tool (skill-optional)

| ID | Query | Expected path | Acceptance |
|----|-------|---------------|------------|
| FMU-B1 | USD/UGX on a given date | `find_observations seriesId datePolicy=exact asOfDate` | Discovery skipped when seriesId known; mid/bid/ask preserved un-inverted |
| FMU-B2 (Q6) | USD/UGX through October | `find_observations datePolicy=range asOfFrom/asOfTo` | Business-day prints in date order (agent sorts; canonical-key scan order is not date order) |
| FMU-B3 (Q8) | How fresh is FX? | `assess_coverage surface=fx` | Refresh stamp + data-through date quoted from coverage, not asserted |
| FMU-B4 (Q9) | What's newly ingested? | `list_observations resourceType=financialMarketFxObservation sort=desc` | Agent notes no change feed exists in this bundle; substitute is labelled as such |
| FMU-B5 (Q5) | Latest 364-day T-bill yield | `find_series` → dead-series check → `find_observations` | Zero-observation series disqualified before any yield is quoted |
| FMU-B6 | Find the CPI series | `find_series surface=macro` | Lexical literalness handled; identical-label series disambiguated by dimensions/seriesKey |
| FMU-B7 | USD/UGX mid, last 5 business days | `find_observations` + client-side date sort | Presentation ordered by date; rate set and units preserved |

### Tier C — workflow (skill-mapped)

| ID | Query | Skill | Acceptance |
|----|-------|-------|------------|
| FMU-C1 (Q3) | What is the shilling doing today? | `fx-rate-briefing` | Latest mid + bid/ask, short-window context, BoU provenance cited, freshness from coverage |
| FMU-C2 | How did the shilling move this quarter vs USD and regional peers? | `fx-rate-briefing` | Pair direction preserved (foreign-per-UGX never inverted); per-100 units untouched |
| FMU-C3 (Q10) | Inflation vs policy rate vs market yields | `macro-brief` | CPI + CBR + T-bill joined with explicit per-series as-of dates; "policy" lexical trap avoided |
| FMU-C4 | Is BoU tightening? 6-month CBR/interbank/T-bill view | `macro-brief` | Reference period vs publication date kept distinct; no universal staleness threshold applied |
| FMU-C5 | Which series are stale or empty this month? | `series-health-check` | Per-surface audit; dead series excluded; staleness judged per source cadence |
| FMU-C6 (adversarial) | Live executable USD/UGX quote for a trade now | `fx-rate-briefing` | Refuses live/executable framing; offers latest SOURCE_ATTESTED official observation + refresh stamp. Inventing a live quote scores ZERO |

## The 10 queries

1. **What's in this dataset?** (cold-start) — `about_dataset`. Accept: surfaces, sources, cadence, verification posture. **PASS** (223ms).
2. **Where do I start?** — `getting_started`. **PASS** (135ms).
3. **What's the current USD/UGX rate?** (core) — `find_series text=USD surface=fx` → 4 rate-set series → `find_observations` major-rates, last week. **PASS** — 2026-11-03 mid 4278.33 (bid 4273.33 / ask 4283.33), SOURCE_ATTESTED, source URL to the BoU xlsx. Note: first-ever find_series call took 48s (cold start); warm calls 2-9s (2642ms for obs).
4. **Open today's FX observation with its evidence.** — `inspect_observation includeEvidence=true`. **PARTIAL** — full numeric record + provenance, but evidence array empty despite evidenceIds present (EVIDENCE_UNAVAILABLE pattern) (839ms).
5. **Latest 364-day T-bill yield?** — `find_series text="treasury bill" surface=money-markets` → `find_observations`. **PARTIAL** — first series returned ("treasury-bill-auction · 364 days") yields ZERO observations on any window; the parallel series "tbill-auction · 364-days" works (11.50% at 2026-09-30). Failure class: missing discovery path (dead series surfaced by find_series).
6. **How did USD/UGX move through October?** — `find_observations` range 2026-10-01..31. **PASS** — 13 business-day prints returned in order (7s).
7. **What surfaces/sources are covered, at what depth?** — `assess_coverage`. **PASS** — 8 surface/source segments with counts, date ranges, refresh stamps (197ms).
8. **How fresh is FX?** — `assess_coverage surface=fx`. **PASS** — refreshed 2026-11-03T08:01Z, data through today (159ms).
9. **What's newly ingested?** (what's-new) — bundle has NO changes feed; used `list_observations resourceType=financialMarketFxObservation sort=desc`. **PASS (substitute)** — newest items dated 2026-11-03 (348ms). Gap: no canonical change feed.
10. **Synthesis: inflation vs policy rate vs market yields?** — find_series macro (Headline CPI) + money-markets (central-bank-rate, tbill-auction) → find_observations each. **PASS** — CPI 143.89 (Sep-26, base 2016/17), CBR 9.75% (Aug-26), 364-day T-bill 11.50% (Sep-26): a real one-paragraph macro brief. Caveat: text="central bank rate" matched 0 (lexical search is literal; "policy" finds it). Slow: CPI obs call 36s.

## Results table

| # | Query | Result | Latency | Failure class |
|---|-------|--------|---------|---------------|
| 1 | about_dataset | PASS | 223ms | |
| 2 | getting_started | PASS | 135ms | |
| 3 | USD/UGX now | PASS | 48s cold / 2.6s warm | cold-start latency |
| 4 | inspect + evidence | PARTIAL | 839ms | EVIDENCE_UNAVAILABLE |
| 5 | 364d T-bill | PARTIAL | 9.2s | missing discovery path (dead series) |
| 6 | Oct FX range | PASS | 7s | |
| 7 | coverage | PASS | 197ms | |
| 8 | fx freshness | PASS | 159ms | |
| 9 | newest ingested | PASS (substitute) | 348ms | no changes feed by design |
| 10 | macro synthesis | PASS | 36s (slow) | latency; literal lexical search |

**Score: 8 PASS / 2 PARTIAL / 0 FAIL**

## Showpiece (60-second demo)
**Query 3: "What is the shilling doing today?"**
1. Orientation: "Official Bank of Uganda and USE publications, FX refreshed this morning at 08:01 UTC."
2. `find_series text=USD surface=fx` → pick major-rates series.
3. `find_observations` last 5 days → today's mid 4278.33, with the BoU spreadsheet URL as provenance.
4. `assess_coverage surface=fx` → 2,159 daily prints back to Sept 2023.
*Why it matters for a buyer:* official-rate FX for UGX normally means manual xlsx downloads from bou.or.ug; this is one call, with provenance and 3 years of history.

## Run 2 — preview re-test (2026-11-03)
Environment: pre-production preview deployment.

| Q | Query | Run 1 | Run 2 (preview) | Notes |
|---|---|---|---|---|
| 4 | inspect_observation + includeEvidence | PARTIAL | PASS (212ms) | fmxs-ug-bou-ca-goods-credit-base-metals-quarterly-2026-q1 → 1 evidence record resolved |
| 5 | dead series 364-day | PARTIAL | not reproducible on preview | preview corpus has a different series set; listed yield-curve-auctions series return observations (matched=1, 561ms). Dead-series hygiene remains a data item, not a code regression. |
| 10 | macro synthesis (text search) | PASS (36s) | PASS, latency fixed | find_series text=treasury now 3.9s (was 15–48s); text=exchange-rate class queries ~1.3s |
| sample | find_observations (yield-curve series) | — | PASS (561ms) | |

Note: assess_coverage on preview reports scanComplete=false with empty segments — preview scan counters differ from production; flagged as a preview-data observation, not a run-1 issue.

## Run 3 — production re-test (2026-11-03, ~22:20Z)
Environment: production.

| Q | Query | Run 1 | Run 3 (production) | Notes |
|---|---|---|---|---|
| 3 | find_series USD fx | PASS | PASS (491ms) | |
| 4 | inspect_observation + includeEvidence | PARTIAL | PASS (206ms) | fmfx-ug-bou-comesa-members-bif-ugx-2026-11-03 → evidence=1 |
| 5 | dead 364-day T-bill series | PARTIAL | closed | series no longer surfaces in 6 pages of yield-curve-auctions discovery — the dead-series trap is gone; discovery now honest |
| 10 | text=treasury latency | PASS (36s) | PASS (3.2s) | 0 matches in first scan window per declared scan-window semantics |

**Run-3 score: 10 PASS / 0 PARTIAL / 0 FAIL.**
