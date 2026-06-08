# GitHub Kanban Board — COS Financial Lakehouse V2

## Project: Financial Lakehouse V2 - AIDLC
## Owner: Narmatha
## Last Updated: 2026-06-08

---

## 📋 Todo

| # | Title | Priority | Epic |
|---|---|---|---|
| 11 | Confirm PVO inventory with Oracle team | P1 | Production |
| 12 | Configure OCI-to-S3 data transfer | P1 | Production |
| 13 | Validate pipeline with real Oracle data | P1 | Production |
| 14 | Connect Power BI to Gold layer | P1 | Production |
| 15 | Deploy to production environment | P1 | Production |
| 16 | Onboard Procurement PVO | P2 | Future |
| 17 | Add CI/CD pipeline | P2 | Future |
| 18 | Add quarantine alerting | P2 | Future |

---

## 🔄 In Progress

| # | Title | Priority | Epic |
|---|---|---|---|
| — | _No items currently in progress_ | — | — |

---

## ✅ Done

| # | Title | Priority | Epic |
|---|---|---|---|
| 1 | Deploy Terraform infrastructure | P0 | Infrastructure |
| 2 | Create Iceberg tables via Athena DDL | P0 | Infrastructure |
| 3 | Implement raw CSV ingestion (Bronze) | P0 | Bronze |
| 4 | Implement all Silver transformations | P0 | Silver |
| 5 | Implement schema drift & evolution | P0 | Silver |
| 6 | Implement star schema (Gold) | P0 | Gold |
| 7 | Create local pytest suite (34 tests) | P0 | Testing |
| 8 | AWS integration testing | P0 | Testing |
| 9 | Project documentation & diagrams | P1 | Docs |
| 10 | AIDLC lifecycle documentation | P1 | Docs |

---

## Progress Metrics

| Metric | Value |
|---|---|
| Total items | 18 |
| Done | 10 (56%) |
| In Progress | 0 (0%) |
| Todo | 8 (44%) |
| P0 items remaining | 0 |
| P1 items remaining | 5 |
| P2 items remaining | 3 |

---

## Dependencies

```
#11 (Oracle PVO) ──→ #12 (OCI→S3) ──→ #13 (Real data) ──→ #15 (Prod deploy)
                                                          ↗
                                    #14 (Power BI) ──────┘
```

#16, #17, #18 are independent — can be done in any order after #15.
