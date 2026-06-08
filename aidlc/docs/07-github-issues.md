# AIDLC Stage 6: GitHub Issues

## Project: COS Financial Lakehouse V2
## Repository: NarmathaGovindaraj/financial-lakehouse-aidlc

---

## Completed Issues (Done)

### Issue #1: [INFRA] Deploy Terraform infrastructure
- **Assignee:** Narmatha
- **Labels:** `P0`, `infrastructure`
- **Description:** Deploy all AWS resources via Terraform: S3 bucket, Glue databases, Glue jobs, IAM role, Athena workgroup, CloudWatch logs.
- **Acceptance Criteria:**
  - [x] 27 resources created
  - [x] All tagged with org standards
  - [x] Zero impact on V1 resources

### Issue #2: [INFRA] Create Iceberg tables
- **Assignee:** Narmatha
- **Labels:** `P0`, `infrastructure`
- **Description:** Create Silver and Gold Iceberg tables via Athena DDL.

### Issue #3: [BRONZE] Implement raw CSV ingestion job
- **Assignee:** Narmatha
- **Labels:** `P0`, `bronze`

### Issue #4: [SILVER] Implement all Silver transformations
- **Assignee:** Narmatha
- **Labels:** `P0`, `silver`

### Issue #5: [SILVER] Implement schema drift detection & evolution
- **Assignee:** Narmatha
- **Labels:** `P0`, `silver`

### Issue #6: [GOLD] Implement star schema (dims + fact)
- **Assignee:** Narmatha
- **Labels:** `P0`, `gold`

### Issue #7: [TEST] Create local pytest suite
- **Assignee:** Narmatha
- **Labels:** `P0`, `testing`

### Issue #8: [TEST] AWS integration testing
- **Assignee:** Narmatha
- **Labels:** `P0`, `testing`

### Issue #9: [DOCS] Create project documentation
- **Assignee:** Narmatha
- **Labels:** `P1`, `documentation`

### Issue #10: [DOCS] Create AIDLC artifacts
- **Assignee:** Narmatha
- **Labels:** `P1`, `documentation`

---

## Open Issues (Todo)

### Issue #11: [PROD] Confirm PVO inventory with Oracle team
- **Labels:** `P1`, `production`

### Issue #12: [PROD] Configure OCI-to-S3 data transfer
- **Labels:** `P1`, `production`

### Issue #13: [PROD] Validate pipeline with real Oracle data
- **Labels:** `P1`, `production`

### Issue #14: [PROD] Connect Power BI to Gold layer
- **Labels:** `P1`, `production`

### Issue #15: [PROD] Deploy to production environment
- **Labels:** `P1`, `production`

### Issue #16: [FUTURE] Onboard Procurement PVO
- **Labels:** `P2`, `future`

### Issue #17: [FUTURE] Add CI/CD pipeline
- **Labels:** `P2`, `future`

### Issue #18: [FUTURE] Add quarantine alerting
- **Labels:** `P2`, `future`
