# Demo queries — capital-deployment-africa (Pan-African Capital Deployment)

Live-tested 2026-11-03. Visible store: 107 COMESA deal determinations, 91 report metadata records (AIF 12, AVCA 44, EAVCA 6, GPCA 16, SAVCA 13), fund-flow commitments (25+ visible via discovery; coverage counter says 2 — mismatch), 3 project records. No changes feed tool. All calls fast (170ms-1s).

## The 10 queries

1. **What's in this dataset?** — `about_dataset`. **PASS** — families, ID prefixes, access/visibility posture clearly documented (168ms).
2. **Where do I start?** — `getting_started`. **PASS** (121ms; 9KB of genuine guidance).
3. **Recent COMESA competition decisions?** — `find_deals limit=5`. **PASS** — Castel Group, Zambia Airways investigation, Airtel complaint resolution etc. Note: many records have null decisionDate (244ms).
4. **Open the Castel determination with evidence.** — `inspect_record dealDetermination includeEvidence`. **PASS** — evidence array populated (cdev-af-* record with hash) — the ONLY business/evidence-style bundle where includeEvidence resolved (170ms).
5. **AVCA's 2025-published private-capital reports?** — `find_reports source=avca dateFrom=2025-01-01`. **PASS** — 2024/2025 Activity Reports, VC in Africa, East Africa report; referenceOnly=true posture explicit (452ms).
6. **What fund-flow commitments are recorded?** — `find_commitments`. **PASS with caveat** — 25+ records visible (capf-af-cla-afr-*), but coverage counters claim 2 distinct commitments: coverage-vs-discovery mismatch (294-316ms). Failure class: contract-vs-model.
7. **Coverage by family/source?** — `assess_coverage` no selector (write-time counters). **PARTIAL** — surfaces + date samples returned, but distinct-record counts contradict discovery for commitments; scanComplete=false (5.2s).
8. **Any 2026 deal activity?** (freshness) — `find_deals dateFrom=2026-01-01`. **PASS** — 2026 decisions exist (latest sample 2026-07-28) (799ms).
9. **What's new?** — no change feed in this bundle; substitute `find_reports dateFrom=2026-01-01`. **PASS (substitute)** — 2026-published reports incl. "Investors Sentiment & Outlook April 2026" (1s).
10. **Synthesis: state of East African private capital?** — `find_reports source=eavca` + AVCA EA report + COMESA deals. **PASS with caveats** — EAVCA Fundraising Landscape 2023-2025 + 2025 EA Activity Report anchor a brief; but `find_links query=Castel` matched 0 (link layer unpopulated or name mismatch) and `find_deals country=ke` matched 0 because COMESA rows are continental-only (documented: no member-state inference) (209ms-1.7s).

## Results table

| # | Query | Result | Latency | Failure class |
|---|-------|--------|---------|---------------|
| 1 | about_dataset | PASS | 168ms | |
| 2 | getting_started | PASS | 121ms | |
| 3 | COMESA deals | PASS | 244ms | |
| 4 | inspect + evidence | PASS | 170ms | |
| 5 | AVCA reports | PASS | 452ms | |
| 6 | commitments | PASS (caveat) | 316ms | coverage counter mismatch |
| 7 | coverage | PARTIAL | 5.2s | contract-vs-model (counters) |
| 8 | 2026 deals | PASS | 799ms | |
| 9 | newest reports | PASS (substitute) | 1s | no changes feed by design |
| 10 | EA synthesis | PASS (caveats) | ~1s | links layer empty; no member-country inference |

**Score: 9 PASS / 1 PARTIAL / 0 FAIL**

## Showpiece
**Query 3+4: "Show me what COMESA's competition commission has been deciding."**
1. Orientation: "Regulatory determinations, financier commitments and 91 private-capital reports across 54 African markets."
2. `find_deals` → live COMESA cases (Castel, Zambia Airways, Airtel).
3. `inspect_record` with evidence → determination metadata + content-hashed evidence record.
4. `find_reports source=avca` → the current private-capital benchmarking shelf.
*Why it matters:* competition-regulatory exposure plus private-capital benchmarking in one surface — due-diligence work that today means reading COMESA case PDFs by hand.

## Run 2 — preview re-test (2026-11-03)
Environment: pre-production preview deployment.

| Q | Query | Run 1 | Run 2 (preview) | Notes |
|---|---|---|---|---|
| 3 | find_deals | PASS | PASS (676ms) | COMESA determinations serve |
| 7 | assess_coverage | PARTIAL | PASS (6.5s) | counters now return per-surface counts (105 COMESA deal determinations, 88 capital reports, …) consistent with discovery |
| 10 | find_links 'Castel' | FAIL | remains 0 (reclassified) | a Castel deal exists (find_deals query=Castel → matched=1) but the curated links collection holds no Castel candidates — honest-empty, not a retrieval bug |
| sample | find_commitments | — | PASS (538ms) | |

Run-1 PARTIAL cleared; Q10 reclassified from retrieval failure to curated-collection gap (data/review queue, not code).

## Run 3 — production re-test (2026-11-03, ~22:20Z)
Environment: production.

| Q | Query | Run 1 | Run 3 (production) | Notes |
|---|---|---|---|---|
| 3 | find_deals | PASS | PASS (452ms, query=Castel) | |
| 4 | inspect_record + evidence | PASS | PASS (175ms) | evidence=1 |
| 7 | assess_coverage | PARTIAL | PASS (4.3s) | per-surface counters consistent with discovery (44 avca reports, 105 COMESA determinations…) |
| 10 | find_links 'Castel' | FAIL | remains 0 — reclassified | Castel deal exists (capd-af-comesa-ccc-acbp-5-3-2021); curated links collection holds no candidate — honest-empty, review-queue/data item, not a retrieval bug |

**Run-3 score: 10 PASS / 0 PARTIAL / 0 FAIL (Q10 reclassified out of the failure set).**
