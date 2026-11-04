# Demo queries — land-property-ug (Uganda Land & Property)

Live-tested 2026-11-03. Data state: 253 records — gazette land-title notices 17 (issues 2026-10-16..30, refreshed today 07:02Z), UBOS CIPI 30 (monthly, through 2026-04), RPPI 11 (quarterly, through 2026-Q2), UPC market snapshots 58 (through 2026-Q4). All surfaces cadence-classified stale=false. Preview-only registration.

## The 10 queries

1. **What's in this dataset?** — `about_dataset`. **PASS** — three families with clear semantics (append-mostly notices, in-place index/snapshot revision), 36-month rolling window stated (158ms).
2. **Where do I start?** — `getting_started`. **PASS** (153ms).
3. **Latest gazette land-title notices?** — `find_notices range 2026-10-01..11-03`. **PASS** — mailo/leasehold special-certificate notices with plot + block refs; cross-issue republication visible (same plot re-noticed 10-16 and 10-23) (999ms).
4. **Open a notice with evidence.** — `inspect_record ltn-ug-2026-10-16-leasehold-...`. **PASS** — 4 evidence records resolved (lpev-ug-gazette-notices-*), verification current (280ms).
5. **How has the national residential price index moved?** — `find_index_points series=rppi-national range 2025+`. **PASS** — 109.54 (2025q1) → 121.07 (2026q1) → 119.64 (2026q2): a real trend line (945ms).
6. **Kampala-area house asking prices, latest quarters?** — `find_market_snapshots area=central-region houses buy`. **PASS** — 2026q3 + 2026q4 snapshots; q4 median UGX 580m across 234 listings, with QoQ change fields (1.5s).
7. **Coverage by family/source?** — `assess_coverage`. **PASS** — 4 segments with cadence-aware stale flags (1.1s).
8. **How fresh are the notices?** — gazette refreshed 2026-11-03T07:02Z, within the 48h cadence; RPPI/CIPI/UPC all cadence-green. **PASS**.
9. **What changed since 1 October?** — `review_changes since=2026-10-01T00:00:00Z`. **PASS** — 10 change records + working pagination. Friction: date-only "2026-10-01" rejected by pattern though description suggests it; needs full timestamp (contract-vs-model, minor) (240ms).
10. **Synthesis: is Kampala property heating or cooling?** — RPPI q/q dip (121.07→119.64) + CIPI construction-cost climb (127.87 Jan → 130.34 Apr) + UPC q4 median UGX 580m + notice flow. **PASS** — a defensible market brief in 4 calls.

## Results table

| # | Query | Result | Latency | Failure class |
|---|-------|--------|---------|---------------|
| 1 | about_dataset | PASS | 158ms | |
| 2 | getting_started | PASS | 153ms | |
| 3 | Oct notices | PASS | 999ms | |
| 4 | inspect + evidence | PASS | 280ms | |
| 5 | RPPI trend | PASS | 945ms | |
| 6 | market snapshots | PASS | 1.5s | |
| 7 | coverage | PASS | 1.1s | |
| 8 | freshness | PASS | (in 7) | |
| 9 | review_changes | PASS | 240ms | date-only `since` rejected (minor) |
| 10 | market synthesis | PASS | ~1s | |

**Score: 10 PASS / 0 PARTIAL / 0 FAIL** — the strongest bundle tested. Minor notes: UPC snapshot records carry empty sourceUrls; review_changes date-only input rejected.

## Showpiece
**Query 10: "Is the Kampala property market heating or cooling?"**
1. Orientation: "Gazette title notices, UBOS price indices, and quarterly asking-price snapshots — all cadence-green today."
2. `find_index_points rppi-national` → official index: up to 121.1 then a 2026-Q2 dip.
3. `find_market_snapshots central-region houses buy` → asking median UGX 580m, 234 listings, QoQ change.
4. `find_notices` → live statutory notice flow as market-activity texture.
5. `assess_coverage` → freshness proof.
*Why it matters:* no other single surface joins Uganda's official property indices, statutory title flow, and market asking prices — valuers and lenders currently assemble this by hand.

## Run 2 — preview re-test (2026-11-03)
Environment: pre-production preview deployment.

| Q | Query | Run 1 | Run 2 (preview) | Notes |
|---|---|---|---|---|
| 5 | rppi-national range | PASS | PASS (2.3s) | 2026q1 point serves |
| 9 | review_changes | PASS | PASS (unchanged path) | date-only `since` still rejected with VALIDATION_ERROR (S4 — out of scope this deploy, "review and schedule"); full ISO datetime accepted as before |

Bundle stays 10/0/0 on the ISO-datetime path; date-only acceptance remains the only polish item.

## Run 3 — production re-test (2026-11-03, ~22:20Z)
Environment: production.

| Q | Query | Run 1 | Run 3 (production) | Notes |
|---|---|---|---|---|
| 5 | rppi-national range | PASS | PASS (1.2s) | 2026q1 point serves |
| 9 | review_changes | PASS | PASS (1.4s) | terminates, complete=true, honest-empty window. Date-only `since` still rejected (S4 — connector schema pattern; scheduled, out of scope this deploy) |

**Run-3 score: 10 PASS / 0 PARTIAL / 0 FAIL maintained.**
