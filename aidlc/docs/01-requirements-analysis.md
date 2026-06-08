# AIDLC Stage 1: Requirements Analysis

## Project: COS Financial Lakehouse V2

---

## Requirements Registry

| REQ | Category | Summary |
|---|---|---|
| REQ-001 | Business Objective | Centralized, automated financial lakehouse replacing manual processes |
| REQ-002 | Stakeholders | CFO, Department Heads, Finance Analysts, Auditors |
| REQ-003 | Data Freshness | Daily refresh sufficient, no real-time |
| REQ-004 | Scope (Phase 1) | Budgetary Control only, establish end-to-end pattern |
| REQ-005 | Scope (Future) | Procurement, Grants, AP, Payroll in later phases |
| REQ-006 | Data Source | Oracle Fusion → BICC → OCI → AWS S3 |
| REQ-007 | Ingestion Pattern | Batch, daily CSV delivery |
| REQ-008 | Dependency | OCI→S3 transfer is external to pipeline |
| REQ-009 | Data Quality | Handle NULLs, duplicates, missing amounts, invalid formats |
| REQ-010 | Quarantine | Isolate bad records, don't fail pipeline |
| REQ-011 | CDC | MERGE/upsert, latest wins |
| REQ-012 | Schema | Auto-detect drift, evolve Iceberg tables |
| REQ-013 | Profiling | Source profiling in Phase 1 to confirm rules |
| REQ-014 | Retention | Multi-year, no auto-deletion |
| REQ-015 | Audit Trail | Full lineage Bronze → Silver → Gold |
| REQ-016 | Compliance | Design for GASB/CAFR (specifics TBD) |
| REQ-017 | Versioning | Iceberg snapshots + Bronze raw files |
| REQ-018 | Cost | Serverless-first, balanced approach |
| REQ-019 | Cost Visibility | Tagging and monitoring |
| REQ-020 | Scalability | Reusable pattern for future PVOs |
| REQ-021 | KPIs | Budget, Actual, Funds Available, Encumbrance, Burn Rate, Variance, QTD/YTD |
| REQ-022 | Drill-Down | City → Department → Fund → Account → Budget Category |
| REQ-023 | Future KPIs | Procurement, Grants, AP, Payroll metrics in future phases |
| REQ-024 | Definition of Done | Production-ready with real Oracle data, Power BI connected, business users validating |
| REQ-025 | Not a POC | Phase 1 is production delivery, not proof-of-concept |
