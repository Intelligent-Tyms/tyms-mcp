# Demo queries — financial-markets-ke (Kenya Financial Markets)

Live-tested 2026-11-03. Data state: production lane since 2026-10-12; FX 694 obs through 2026-11-03 (today), equities 884 (through 11-02), bonds 354, money-markets 191, external-sector 599, macro 228 (CBK) + 56 (KNBS). Sources: CBK, KNBS, NSE.

## The 10 queries

1. **What's in this dataset?** — `about_dataset`. **PASS** (147ms).
2. **Where do I start?** — `getting_started`. **PASS** (115ms).
3. **What's the current USD/KES rate?** — `find_series text=USD surface=fx` → `get_observations`. **PASS** — 2026-11-03 indicative mid 129.97 (CBK), provenance URL centralbank.go.ke/forex. Latency: find 17s, obs 7s.
4. **Open today's FX observation with evidence.** — `inspect_observation includeEvidence=true`. **PARTIAL** — record complete, evidence array empty despite evidenceIds (EVIDENCE_UNAVAILABLE pattern) (1.2s).
5. **Absa Bank's recent NSE closes?** — `find_series text=Absa surface=equities` → `get_observations`. **PASS** — 5 trading-day observations 26-30 Oct with per-day records. Latency warning: find_series text search took 42s.
6. **How did USD/KES move through October?** — range 2026-10-01..31. **PASS** — 16 prints, 129.94→129.95 (5.9s).
7. **Coverage by surface/source?** — `assess_coverage`. **PASS** — 7 segments with counts/ranges/refresh (184ms).
8. **How fresh is FX?** — refreshed 2026-11-03T08:34Z, data through today. **PASS** (in Q7 call).
9. **What's newly ingested?** — no changes feed in this bundle; `list_observations resourceType=financialMarketEquityObservation sort=desc` → 2026-11-02 NSE closes. **PASS (substitute)** (167ms).
10. **Synthesis: Kenyan rates & inflation picture?** — 91-day T-bill series (8.7498% at 2026-11-02) + KNBS CPI series (core index through 2026-09) + FX 129.97. **PASS** as a brief. Caveats: (a) no central-bank-rate series in KE money-markets (discount window + interbank + T-bills only), (b) the three KNBS CPI series share one identical label and differ only in dimensions (seriesKey cpi-core-index / cpi-core-yoy / cpi-food-yoy) — label ambiguity, (c) CPI obs call took 33s.

## Results table

| # | Query | Result | Latency | Failure class |
|---|-------|--------|---------|---------------|
| 1 | about_dataset | PASS | 147ms | |
| 2 | getting_started | PASS | 115ms | |
| 3 | USD/KES now | PASS | 17s + 7s | latency |
| 4 | inspect + evidence | PARTIAL | 1.2s | EVIDENCE_UNAVAILABLE |
| 5 | Absa closes | PASS | 42s find (slow) | latency |
| 6 | Oct FX range | PASS | 5.9s | |
| 7 | coverage | PASS | 184ms | |
| 8 | freshness | PASS | (in 7) | |
| 9 | newest ingested | PASS (substitute) | 167ms | no changes feed by design |
| 10 | rates+inflation synthesis | PASS | 33s (slow) | latency; label ambiguity; CBR series absent |

**Score: 9 PASS / 1 PARTIAL / 0 FAIL**

## Showpiece
**Query 3: "Where is the Kenyan shilling today?"**
1. Orientation: "Official CBK, KNBS and NSE publications; FX refreshed this morning."
2. `find_series USD/fx` → indicative USD/KES series.
3. `get_observations` → 129.97 mid today, 129.95 Friday — plus the CBK source URL.
4. `assess_coverage` → 7 surfaces with live refresh stamps.
*Why it matters:* CBK indicative rates + NSE closes + KNBS inflation in one query surface replaces three manual source checks.

## Run 2 — preview re-test (2026-11-03)
Environment: pre-production preview deployment.

| Q | Query | Run 1 | Run 2 (preview) | Notes |
|---|---|---|---|---|
| 3 | find_series (fx) | PASS | PASS (1.3s) | GBP/KES etc. |
| 4 | inspect_observation + includeEvidence | PARTIAL | PASS (188ms) | fmfx-ke-cbk-indicative-gbp-kes-2026-10-08 → 1 evidence record resolved |
| 10 | synthesis (was 33s caveat) | PASS | PASS, latency fixed | find_series now ~1.3s |

Run-1 PARTIAL cleared on the fixed build.

## Run 3 — production re-test (2026-11-03, ~22:20Z)
Environment: production.

| Q | Query | Run 1 | Run 3 (production) | Notes |
|---|---|---|---|---|
| 3 | find_series USD fx | PASS | PASS (4.0s) | USD/KES indicative, today's print |
| 4 | inspect_observation + includeEvidence | PARTIAL | PASS (157ms) | fmfx-ke-cbk-indicative-usd-kes-2026-11-03 → evidence=1 |
| 10 | synthesis latency | PASS (33s caveat) | PASS (2.9s) | text=inflation scan-window semantics |

**Run-3 score: 10 PASS / 0 PARTIAL / 0 FAIL.**
