# AIDLC Executive Walkthrough — 5 Minute Presentation

## Project: COS Financial Lakehouse V2
## Presenter: Narmatha

---

## Slide 1: The Problem (30 seconds)

> "The City of Springfield manages a $367 million annual budget across 10+ departments using Oracle Fusion. Today, producing a budget vs actual report requires manual extraction, reconciliation across multiple sources, and spreadsheet-based analysis. This takes days and is error-prone.
>
> We built a Financial Lakehouse on AWS that automates this entire pipeline — from Oracle extraction to Power BI dashboards — running daily with zero manual intervention."

---

## Slide 2: AIDLC Process (30 seconds)

| Stage | Delivered | Artifacts |
|---|---|---|
| Requirements | 25 requirements captured | 01-requirements-analysis.md |
| User Stories | 24 stories with acceptance criteria | 02-user-stories.md |
| Architecture | 8 architecture decisions with trade-offs | 03-architecture-design.md |
| Functional Design | Complete data flow, schemas, rules | 04-functional-design.md |
| Testing | 34 automated tests, traceability matrix | 05-testing-strategy.md |
| Implementation | 63 tasks tracked, 71% complete | 06-implementation-plan.md |

---

## Slide 3: Architecture (1 minute)

| Layer | What it does | Technology |
|---|---|---|
| Bronze | Stores raw Oracle CSV as-is (audit trail) | S3 |
| Silver | All processing: validate, dedup, CDC, schema evolution | Glue + Iceberg |
| Gold | Star schema for Power BI | Glue + Iceberg |

Key decisions:
- Iceberg for MERGE INTO, time travel, and schema evolution
- Glue serverless (no idle cost)
- Terraform for reproducible infrastructure (27 resources in 12 seconds)

---

## Slide 4: Live Demo Results (1.5 minutes)

- 7 raw Oracle rows → Bronze (raw CSV stored)
- Silver: 1 quarantined, 1 deduped, NULLs handled, fiscal year derived
- Result: 5 clean rows in Silver Iceberg
- Gold: 5 departments × budget metrics in star schema
- Schema drift: TaskId column adapted automatically
- Time travel: 2 Iceberg snapshots visible
- Total pipeline cost: $5.28 per run. Idle cost: $0.

---

## Slide 5: Testing & Quality (30 seconds)

| Category | Tests | What it proves |
|---|---|---|
| Bronze | 4 | Raw data preserved |
| Silver | 12 | All transformation logic correct |
| Iceberg | 4 | Snapshots, time travel, rollback |
| Gold | 4 | Star schema and aggregations |
| Reconciliation | 5 | No money lost between layers |
| End-to-End | 2 | Full pipeline works as a unit |
| Bookmarks | 3 | Incremental processing works |

---

## Slide 6: Project Status (30 seconds)

| Priority | Done | Todo |
|---|---|---|
| P0 (Core) | 40/40 | 0 |
| P1 (Production) | 5/16 | 11 |
| P2 (Future) | 0/7 | 7 |

Next steps: Oracle PVO workshop → OCI-to-S3 transfer → Real data validation → Power BI → Production deploy

---

## Slide 7: What Makes This Different

1. **Self-healing pipeline** — Schema drift doesn't break anything
2. **Instant recovery** — Iceberg rollback reverts bad loads in seconds
3. **Complete traceability** — Every requirement → story → test → code

---

## Quick Q&A Answers

- **Why Iceberg?** MERGE INTO, time travel, schema evolution. Plain Parquet can't do any of these.
- **Monthly cost?** ~$160/month running daily. $0 when idle.
- **Time to add new PVO?** 2-3 days (framework is built).
- **Production-ready?** Pipeline code yes. Pending: real Oracle data + Power BI.
