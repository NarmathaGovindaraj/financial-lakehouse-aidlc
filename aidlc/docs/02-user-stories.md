# AIDLC Stage 2: User Stories

## Project: COS Financial Lakehouse V2

---

## Epic 1: Data Ingestion (Bronze Layer)

### US-01: Raw Data Ingestion
**As a** Data Engineer, **I want** Oracle BICC CSV extracts stored in Bronze exactly as received, **So that** we have an unmodified audit trail.

**Acceptance Criteria:**
- [ ] CSV files copied to `bronze/{table}/load_date=YYYY-MM-DD/`
- [ ] Original filename preserved
- [ ] No transformation applied
- [ ] File content byte-for-byte identical to source
- [ ] Job runs daily on schedule
- [ ] Job bookmark prevents reprocessing

### US-02: Incremental File Processing
**As a** Data Engineer, **I want** the Bronze job to only process new files, **So that** we don't waste compute re-processing.

**Acceptance Criteria:**
- [ ] Job bookmark tracks last processed load_date
- [ ] Previously processed dates skipped
- [ ] New load_date files picked up automatically

---

## Epic 2: Data Processing (Silver Layer)

### US-03: Data Validation & Quarantine
**As a** Data Engineer, **I want** records with NULL primary keys quarantined, **So that** bad data is isolated while good data flows through.

**Acceptance Criteria:**
- [ ] NULL `CodeCombinationId` records → quarantine
- [ ] Quarantine includes reason + date
- [ ] Pipeline does NOT fail on bad records

### US-04: Column Standardization
**As a** Data Engineer, **I want** CamelCase columns converted to snake_case, **So that** downstream consumers have consistent naming.

**Acceptance Criteria:**
- [ ] `CodeCombinationId` → `code_combination_id`
- [ ] All Oracle columns renamed consistently

### US-05: Deduplication
**As a** Finance Director, **I want** only the latest version of each budget record, **So that** I see current approved amounts.

**Acceptance Criteria:**
- [ ] Duplicates by natural key (ccid + period)
- [ ] Latest record kept based on last_update_date
- [ ] Older duplicates removed

### US-06: NULL Handling
**As a** Finance Analyst, **I want** NULL numeric values replaced with 0.0, **So that** calculations don't produce NULL results.

**Acceptance Criteria:**
- [ ] NULL actual/encumbrance/commitment/obligation → 0.0
- [ ] Non-NULL values unchanged

### US-07: Fiscal Year Derivation
**As a** CFO, **I want** fiscal year derived from period_name (Jul-Jun), **So that** reports align with COS fiscal calendar.

**Acceptance Criteria:**
- [ ] JUL-2024 → FY2025, JAN-2025 → FY2025
- [ ] All 12 months map correctly

### US-08: Financial Metrics Calculation
**As a** Department Head, **I want** funds_available and burn_rate calculated automatically, **So that** I see remaining budget without manual work.

**Acceptance Criteria:**
- [ ] funds_available = budget - actual - encumbrance
- [ ] burn_rate = (actual + encumbrance) / budget

### US-09: CDC / MERGE INTO Silver
**As a** Data Engineer, **I want** incoming records merged via CDC logic, **So that** existing records are updated and new ones inserted.

**Acceptance Criteria:**
- [ ] New records INSERTED
- [ ] Existing newer records UPDATED
- [ ] Existing older records SKIPPED
- [ ] Operation is atomic

### US-10: Schema Drift Detection
**As a** Data Engineer, **I want** pipeline to detect when Oracle adds new columns, **So that** schema changes don't break processing.

**Acceptance Criteria:**
- [ ] New columns detected by comparing CSV headers
- [ ] Detection logged
- [ ] Pipeline continues (no failure)

### US-11: Schema Evolution
**As a** Data Engineer, **I want** new columns added to Iceberg table automatically, **So that** future data with new fields is captured.

**Acceptance Criteria:**
- [ ] ALTER TABLE ADD COLUMNS executed
- [ ] Old rows show NULL for new columns
- [ ] Table remains queryable

### US-12: Audit Columns
**As an** Auditor, **I want** insert/update timestamps on every record, **So that** I can trace when data entered the lakehouse.

**Acceptance Criteria:**
- [ ] dw_insert_date, dw_update_date, bicc_extract_id, load_date populated

---

## Epic 3: Analytics Model (Gold Layer)

### US-13: Chart of Accounts Dimension
**As a** Finance Analyst, **I want** a dimension mapping CCIDs to department names, **So that** reports show names not IDs.

**Acceptance Criteria:**
- [ ] dim_coa with coa_key, code_combination_id, department_description
- [ ] MERGE logic handles new departments

### US-14: Fiscal Period Dimension
**As a** Finance Analyst, **I want** a fiscal calendar dimension, **So that** I can slice by quarter/month/year.

**Acceptance Criteria:**
- [ ] dim_period with fiscal_year, fiscal_quarter, fiscal_month
- [ ] Q1=Jul-Sep, Q2=Oct-Dec, Q3=Jan-Mar, Q4=Apr-Jun

### US-15: Budgetary Control Fact Table
**As a** CFO, **I want** a fact table joining budget data with dimensions, **So that** Power BI shows budget vs actual by department.

**Acceptance Criteria:**
- [ ] Joins to dim_coa and dim_period
- [ ] No orphan keys
- [ ] SUM(budget) in fact = SUM(budget) in Silver

---

## Epic 4: Iceberg Capabilities

### US-16: Time Travel
**As an** Auditor, **I want** to query past states of the Silver table, **So that** I can verify historical reports.

**Acceptance Criteria:**
- [ ] FOR TIMESTAMP AS OF queries work
- [ ] Snapshots retained 30+ days

### US-17: Rollback & Recovery
**As a** Data Engineer, **I want** to rollback after bad loads, **So that** corrupted data is reverted in seconds.

**Acceptance Criteria:**
- [ ] Rollback restores prior state instantly
- [ ] No data rewrite needed

---

## Epic 5: Infrastructure & Operations

### US-18: Infrastructure as Code
**As an** IT Director, **I want** all resources in Terraform, **So that** environments are reproducible.

**Acceptance Criteria:**
- [ ] terraform apply creates all resources
- [ ] terraform destroy removes cleanly
- [ ] All resources tagged

### US-19: Pipeline Orchestration
**As a** Data Engineer, **I want** jobs chained automatically, **So that** one trigger runs the full pipeline.

**Acceptance Criteria:**
- [ ] Glue Workflow with conditional triggers
- [ ] Bronze → Silver → Gold chaining

### US-20: Cost Tagging & Monitoring
**As an** IT Director, **I want** all resources tagged for cost allocation, **So that** AWS costs are attributable.

**Acceptance Criteria:**
- [ ] 7 mandatory tags on all resources
- [ ] Tags configurable via tfvars

---

## Epic 6: Testing & Quality

### US-21: Local ETL Testing
**As a** Data Engineer, **I want** to run tests locally before deploying, **So that** bugs are caught for free.

**Acceptance Criteria:**
- [ ] pytest + PySpark, no AWS needed
- [ ] 30+ tests pass in < 30 seconds

### US-22: Financial Reconciliation
**As a** Finance Director, **I want** automated checks ensuring no money lost between layers, **So that** I trust the totals.

**Acceptance Criteria:**
- [ ] SUM(budget) Silver = SUM(budget) Gold
- [ ] Row count reconciliation across layers

---

## Epic 7: Reporting

### US-23: Power BI Connection
**As a** Finance Analyst, **I want** Power BI connected to Gold via Athena, **So that** I build dashboards without SQL.

**Acceptance Criteria:**
- [ ] Athena workgroup configured
- [ ] Power BI connects via ODBC

### US-24: Budget vs Actual Dashboard
**As a** CFO, **I want** budget vs actual by department and period, **So that** I identify overspending.

**Acceptance Criteria:**
- [ ] Budget, Actual, Funds Available, Burn Rate per department
- [ ] Filterable by fiscal year/quarter
- [ ] Drill-down: Department → Fund → Account
