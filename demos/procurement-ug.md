# Demo queries — procurement-ug (Uganda Procurement)

Live-tested 2026-11-03. Preview-live. Live tender flow confirmed (open opportunities with Nov-2026 deadlines from NITA-U, NPA, DCIC, MoWE, BoU). assess_coverage counters unavailable; change feed unpopulated; text search slow.

Contract update 2026-11-09: `includeEvidence` defaults to true on `inspect_record`; omission resolves evidence, while `includeEvidence=false` opts out. Canonical structured public IDs infer `resourceType`.

## Tiered eval set (v2, 2026-11-07)

Skill-A/B eval shape per the internal workflow plan §4.
Tier A = simple lookup, **no skill by design** (control group). Tier B = directed
multi-tool, skill-optional. Tier C = workflow, each mapped to exactly one skill.
The original 10 queries below remain the regression core; IDs in parentheses map
old queries into the tiers. Acceptance = machine-checkable first, judge second.

### Tier A — simple lookup (skill: none)

| ID | Query | Expected path | Acceptance |
|----|-------|---------------|------------|
| PUG-A1 (Q1) | What's in this dataset? | `about_dataset` | Sources (PPDA, eGP, BEB, direct) named; no invented freshness objective |
| PUG-A2 (Q2) | Where do I start? | `getting_started` | Points at about_dataset + task tools |
| PUG-A3 (Q3) | What open tenders close after today? | `find_opportunities status=open deadlineAfter=<today>` | ≥1 live tender; `buyerNames` + `submissionDeadline` preserved verbatim |
| PUG-A4 (Q4) | Open tender \<publicId\> with evidence | `inspect_record publicId` (canonical prefix infers resourceType; includeEvidence omitted) | Full record; evidence resolved OR honest `evidence=0`; never invented |
| PUG-A5 (Q7) | Coverage by source/resource? | `assess_coverage` | `counterSnapshotAvailable` + `countQualifier` quoted as returned |

### Tier B — directed multi-tool (skill-optional)

| ID | Query | Expected path | Acceptance |
|----|-------|---------------|------------|
| PUG-B1 (Q5) | Notices mentioning 'road' | `find_notices text=road` | Agent states literal-substring + slow-scan expectation before/while calling |
| PUG-B2 (Q6) | Which buyers are in the data? | `list_buyers` | Per-tender-ref identity caveat surfaced; no deduplicated-entity claim |
| PUG-B3 (Q8) | Newest activity? | `list_notices sort=desc` | Agent states sort is submission-deadline, not publication; no global-latest claim |
| PUG-B4 (Q9) | What changed since \<date\>? | `review_changes since=<RFC3339>` | Pages while `nextCursor` non-null; stops at `pagination.complete=true`; retains `checkpointCursor`; near-present honest-empty |
| PUG-B5 | Who won recent awards? | `list_awards sort=desc` | Current awards with supplier/buyer names exactly as returned |
| PUG-B6 | Trace process \<id\> end to end | `get_process_by_id` → `list_notices/list_awards processId=` | Joins by `processId` only; all IDs verbatim |
| PUG-B7 | Open works tenders from buyer X closing in 30 days | `find_opportunities buyerId+deadlineAfter/deadlineBefore` | `buyerId` obtained from discovery, never guessed; AND-filter semantics stated |

### Tier C — workflow (skill-mapped)

| ID | Query | Skill | Acceptance |
|----|-------|-------|------------|
| PUG-C1 (Q10) | What tenders can I still bid on, and who's winning similar work? | `bidder-pipeline-scan` | Open opps by deadline + recent awards joined; no eligibility advice; freshness quoted from coverage |
| PUG-C2 | Bid-prep brief for tender \<id\> | `bidder-pipeline-scan` | Exact deadline, buyer, evidence resolved; no eligibility inference when date fields absent |
| PUG-C3 | Watch process \<id\>: what changed this week? | `tender-deadline-watch` | Checkpoint-cursor handling correct incl. unchanged-empty-checkpoint = no changes |
| PUG-C4 | Weekly new-tender digest, roads sector | `tender-deadline-watch` | Structured filters first; `text=` only as stated last resort; publication-ordering limit stated |
| PUG-C5 | Which buyers award vehicle-maintenance contracts, who wins? | `buyer-award-history` | Buyer fragmentation handled; awards joined by `buyerId`; suppliers verbatim |
| PUG-C6 (adversarial) | Complete all-time award history for buyer X | `buyer-award-history` | Refuses "complete/all-time" framing; permanent bounded tail + scan window quoted. Confident completeness claim scores ZERO |

## The 10 queries

1. **What's in this dataset?** — `about_dataset`. **PASS** — sources (PPDA, eGP, BEB, MOES/MOWT direct), preview status, no approved freshness objective stated honestly (139ms).
2. **Where do I start?** — `getting_started`. **PASS** (81ms).
3. **What open tenders close after today?** — `find_opportunities status=open deadlineAfter=2026-11-03`. **PASS** — live tenders: NITA-U Last Mile Connectivity (closes 16 Nov), NPA staff medical (10 Nov), BoU X-ray/AC supply, MoWE fuel (4.8s).
4. **Open the NPA medical tender with evidence.** — `inspect_record publicId` (canonical prefix infers resourceType; includeEvidence omitted). **PASS** — 2 evidence records resolved with content hashes (382ms).
5. **Notices mentioning 'road'?** — `find_notices text=road`. **PASS but slow** — 2 hits (Entebbe Airport road toll shade works etc.), 36s substring scan (36s).
6. **Which buyers are in the data?** — `list_buyers limit=5`. **PASS with data-model note** — buyers are per-tender-reference identities (UCAA appears once per procurement ref), not deduplicated entities (306ms).
7. **Coverage by source/resource?** — `assess_coverage` (no args, ppda, uganda-egp). **FAIL** — all counters null (counterSnapshotAvailable=false); uganda-egp probe took 120s and reports sourcePresenceObserved=false. Failure class: contract-vs-model (coverage surface degraded).
8. **What's the newest activity?** (freshness) — `list_notices sort=desc`. **PARTIAL** — sorts by submission deadline, not publication (publication-date/global-latest documented as unsupported); freshest deadlines 2026-10-29, awards current (320ms).
9. **What changed since 2026-11-01?** — `review_changes since=2026-11-01`, followed 6 pages. **FAIL** — 0 records, pagination never completes, no checkpoint cursor. Failure class: continuation-after-bounded-empty (change feed unpopulated) (2.3s for 6 pages).
10. **Synthesis: pipeline for a works/services bidder?** — open opportunities + latest awards (tyres/vehicle repair for MoLHUD). **PARTIAL** — open+awards joins work, but a text-filtered slice (text='works', open) matched 0 despite works-type tenders existing — literal substring over limited fields (6.6s).

## Results table

| # | Query | Result | Latency | Failure class |
|---|-------|--------|---------|---------------|
| 1 | about_dataset | PASS | 139ms | |
| 2 | getting_started | PASS | 81ms | |
| 3 | open opportunities | PASS | 4.8s | |
| 4 | inspect + evidence | PASS | 382ms | |
| 5 | notices 'road' | PASS (slow) | 36s | latency |
| 6 | buyers | PASS (note) | 306ms | buyer identity fragmented per tender ref |
| 7 | coverage | FAIL | 1.5s / 120s | contract-vs-model (counters null) |
| 8 | freshness | PARTIAL | 320ms | no publication-date ordering |
| 9 | review_changes | FAIL | 2.3s | continuation-after-bounded-empty |
| 10 | bidder synthesis | PARTIAL | 6.6s | literal text-search limits |

**Score: 6 PASS / 2 PARTIAL / 2 FAIL**

## Showpiece
**Query 3+4: "What Ugandan government tenders can I still bid on?"**
1. Orientation: "PPDA + eGP tender flow, preview-live, updated continuously."
2. `find_opportunities status=open deadlineAfter=today` → live tenders with buyers and deadlines.
3. `inspect_record` on one → full notice + hashed evidence records.
4. `list_awards sort=desc` → who is winning what, current awards.
*Why it matters:* a bidder's Monday-morning pipeline scan — open tenders by deadline plus recent awards — without logging into the eGP portal. (Demo caveat: avoid coverage and changes tools; run from the per-bundle file.)

## Run 2 — preview re-test (2026-11-03)
Environment: pre-production preview deployment.

| Q | Query | Run 1 | Run 2 (preview) | Notes |
|---|---|---|---|---|
| 7 | assess_coverage | FAIL | PASS (1.4s) | counterSnapshotAvailable=true, countQualifier=write_time_counters — counters self-healed, converged on first call |
| 8 | freshness | PARTIAL | PARTIAL (unchanged) | no publication-date ordering; not a fix target this deploy |
| 9 | review_changes | FAIL | PASS | since=2026-08-01 → 10 records/page at ~0.5s, pagination progresses; near-present windows honest-empty |
| 10 | text='works' | PARTIAL | PASS (3.8s) | matched=5 (was 0); text=road matched=2 in 4.9s (was 36–48s). Text-search latency fix confirmed. |

Equivalent run-2 score on the fixed build: 9 PASS / 1 PARTIAL / 0 FAIL. The run-1 demo caveat (avoid coverage and changes tools) no longer applies on the fixed build.

## Run 3 — production re-test (2026-11-03, ~22:20Z)
Environment: production.

| Q | Query | Run 1 | Run 3 (production) | Notes |
|---|---|---|---|---|
| 3 | open opportunities | PASS | PASS (2.3s) | |
| 4 | inspect_record + evidence | PASS | PASS (222ms) | sampled opportunity honestly reports evidence=0 (no evidence objects for that record) |
| 7 | assess_coverage | FAIL | PASS (1.2s warm) | counterSnapshotAvailable=true, write_time_counters; deploy session saw cold-snapshot convergence at call 9, then ~0.9s steady — ops note, not a defect |
| 8 | freshness | PARTIAL | PARTIAL (unchanged) | no publication-date ordering; backlog item |
| 9 | review_changes | FAIL | PASS (452ms) | 10 records immediately on since=2026-10-01; today-window honest-empty with checkpoint cursor |
| 10 | text='works' | PARTIAL | PASS (5.5s) | matched=4 (was 0 in 36–48s) |

**Run-3 score: 9 PASS / 1 PARTIAL / 0 FAIL.**
