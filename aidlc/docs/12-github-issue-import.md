# GitHub Issue Import — Ready for Creation

## Repository: NarmathaGovindaraj/financial-lakehouse-aidlc
## Total Issues: 18

---

## Issue #1

- **Title:** [INFRA] Deploy Terraform infrastructure
- **Labels:** `P0`, `infrastructure`
- **Priority:** P0
- **Assignee:** NarmathaGovindaraj
- **Status:** Done

**Description:**
Deploy all AWS resources via Terraform for the Financial Lakehouse V2 project.

Resources created:
- S3 bucket: cos-financial-lakehouse-v2
- Glue databases: cos_silver_v2, cos_gold_v2
- Glue jobs: COS-Oracle-to-Bronze-V2, COS-Bronze-to-Silver-V2, COS-Silver-to-Gold-V2
- Glue workflow: COS-Lakehouse-Pipeline-V2 with 3 triggers
- IAM role: cos-lakehouse-glue-etl-role-v2
- Athena workgroup: cos-financial-lakehouse-v2
- CloudWatch log groups (3)

**Acceptance Criteria:**
- [x] 27 resources created successfully
- [x] All resources tagged with mandatory org tags
- [x] Zero impact on existing V1 resources
- [x] terraform plan shows 0 changes after apply

---

## Issue #2

- **Title:** [INFRA] Create Iceberg tables via Athena DDL
- **Labels:** `P0`, `infrastructure`
- **Priority:** P0
- **Assignee:** NarmathaGovindaraj
- **Status:** Done

---

## Issue #3

- **Title:** [BRONZE] Implement raw CSV ingestion job
- **Labels:** `P0`, `bronze`
- **Status:** Done

---

## Issue #4

- **Title:** [SILVER] Implement all Silver layer transformations
- **Labels:** `P0`, `silver`
- **Status:** Done

---

## Issue #5

- **Title:** [SILVER] Implement schema drift detection and evolution
- **Labels:** `P0`, `silver`
- **Status:** Done

---

## Issue #6

- **Title:** [GOLD] Implement star schema (dimensions + fact table)
- **Labels:** `P0`, `gold`
- **Status:** Done

---

## Issue #7

- **Title:** [TEST] Create local PySpark/pytest test suite
- **Labels:** `P0`, `testing`
- **Status:** Done

---

## Issue #8

- **Title:** [TEST] AWS integration testing
- **Labels:** `P0`, `testing`
- **Status:** Done

---

## Issue #9

- **Title:** [DOCS] Create project documentation and diagrams
- **Labels:** `P1`, `documentation`
- **Status:** Done

---

## Issue #10

- **Title:** [DOCS] Create AIDLC lifecycle documentation
- **Labels:** `P1`, `documentation`
- **Status:** Done

---

## Issue #11

- **Title:** [PROD] Confirm PVO inventory with Oracle team
- **Labels:** `P1`, `production`
- **Status:** Todo

---

## Issue #12

- **Title:** [PROD] Configure OCI-to-S3 data transfer
- **Labels:** `P1`, `production`
- **Status:** Todo

---

## Issue #13

- **Title:** [PROD] Validate pipeline with real Oracle data
- **Labels:** `P1`, `production`
- **Status:** Todo

---

## Issue #14

- **Title:** [PROD] Connect Power BI to Gold layer via Athena
- **Labels:** `P1`, `production`
- **Status:** Todo

---

## Issue #15

- **Title:** [PROD] Deploy to production environment
- **Labels:** `P1`, `production`
- **Status:** Todo

---

## Issue #16

- **Title:** [FUTURE] Onboard Procurement PVO
- **Labels:** `P2`, `future`
- **Status:** Todo

---

## Issue #17

- **Title:** [FUTURE] Add CI/CD pipeline (GitHub Actions)
- **Labels:** `P2`, `future`
- **Status:** Todo

---

## Issue #18

- **Title:** [FUTURE] Add quarantine alerting (SNS notification)
- **Labels:** `P2`, `future`
- **Status:** Todo
