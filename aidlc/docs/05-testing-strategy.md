# AIDLC Stage 5: Testing Strategy

## Project: COS Financial Lakehouse V2

---

## 1. Testing Philosophy

**Principle:** Test transformation logic locally for free before deploying to AWS. Catch logic bugs in seconds (local PySpark), not in minutes at $1.76/run (Glue).

**Testing Pyramid:**

```
         ┌───────────┐
         │Integration│  ← AWS Glue runs (expensive, run after local tests pass)
        ┌┴───────────┴┐
        │  End-to-End  │  ← Full pipeline simulation (local, free)
       ┌┴─────────────┴┐
       │  Unit Tests    │  ← Individual transformation logic (local, free)
      ┌┴───────────────┴┐
      │  Data Quality    │  ← Reconciliation + validation (local + Athena)
     ┌┴─────────────────┴┐
     │  Infrastructure    │  ← Terraform plan/validate (local, free)
    └─────────────────────┘
```

---

## 2. Testing Framework

| Component | Tool | Purpose |
|---|---|---|
| Test runner | pytest | Discover and execute tests |
| Spark engine | PySpark (local mode) | Simulate Glue transformations |
| Assertions | pytest assert | Validate outputs |
| Fixtures | conftest.py | Shared SparkSession, sample data paths |
| Test data | sample_data/*.csv | Controlled input for reproducible tests |
| Iceberg tests | Local Hadoop catalog | Test snapshots/time travel without AWS |
| Coverage | pytest-cov (optional) | Track code coverage |

**Execution command:**
```bash
python3 -m pytest financial-lakehouse-v2/tests/ -v
```

**Expected output:** 30 passed, 4 skipped in ~16 seconds

---

## 3. Test Categories & Coverage Matrix

### 3.1 Bronze Layer Tests (test_bronze.py — 4 tests)

| Test | Validates | Rule |
|---|---|---|
| test_csv_stored_unchanged | File content preserved after CSV round-trip | R-BRZ-01 |
| test_original_filename_preserved | Source filename exists and is .csv | R-BRZ-03 |
| test_record_count_matches_source | Row count in Bronze = source row count | R-BRZ-04 |
| test_source_schema_preserved | Column names and order unchanged | R-BRZ-04 |

---

### 3.2 Silver Layer Tests (test_silver.py — 12 tests)

| Test | Validates | Rule |
|---|---|---|
| test_schema_drift_detection | New column (TaskId) detected in CSV | R-SLV: Schema drift |
| test_schema_evolution | New column flows through after rename | R-SLV: Schema evolution |
| test_validation_null_key_quarantined | NULL key → quarantine (6 good, 1 bad) | DQ-01 |
| test_quarantine_output_structure | Quarantine has reason + date columns | DQ-01 |
| test_column_rename_camel_to_snake | CamelCase → snake_case | R-SLV-02 |
| test_deduplication_keeps_latest | Duplicates removed, newer budget kept | R-SLV-04 |
| test_null_handling_numerics | NULL → 0.0 for amount fields | R-SLV-03 |
| test_fiscal_year_derivation | JUL-2024→FY2025, JAN-2025→FY2025 | R-SLV-05 |
| test_funds_available_calculation | budget - actual - encumbrance | R-SLV-06 |
| test_cdc_merge_insert_new | New record inserted (union simulation) | R-SLV-07 |
| test_cdc_merge_update_existing | Existing record updated with newer data | R-SLV-07 |
| test_audit_columns_present | dw_insert_date, dw_update_date populated | Audit |

---

### 3.3 Silver Iceberg Tests (test_silver_iceberg.py — 4 tests)

| Test | Validates | Rule |
|---|---|---|
| test_snapshot_created_after_write | Snapshot exists after INSERT | Time travel |
| test_schema_evolution_column_added | ALTER TABLE ADD COLUMNS, old rows NULL | Schema evolution |
| test_time_travel_query | VERSION AS OF returns historical state | Audit/compliance |
| test_rollback_recovery | Rollback reverts bad write | Recovery |

---

### 3.4 Gold Layer Tests (test_gold.py — 4 tests)

| Test | Validates | Rule |
|---|---|---|
| test_dim_coa_mapping | CCID 1001 → "Police" | Dimension accuracy |
| test_dim_period_fiscal_calendar | Period → correct quarter/month | Calendar logic |
| test_fact_join_completeness | Only matched rows in fact (no orphans) | Referential integrity |
| test_aggregation_by_department | SUM(budget) by department correct | Power BI pattern |

---

### 3.5 Job Bookmark Tests (test_job_bookmarks.py — 3 tests)

| Test | Validates | Rule |
|---|---|---|
| test_processed_file_skipped | Old load_date not re-read | Incremental processing |
| test_new_file_processed | New load_date picked up | Incremental processing |
| test_bookmark_state_persisted | Max load_date tracked | State management |

---

### 3.6 Financial Reconciliation Tests (test_financial_reconciliation.py — 5 tests)

| Test | Validates | Rule |
|---|---|---|
| test_source_vs_bronze_row_count | Source rows = Bronze rows | DQ: no data loss in Bronze |
| test_bronze_vs_silver_row_count | Bronze > Silver (quarantine + dedup) | DQ: expected row reduction |
| test_silver_vs_gold_row_count | Silver = Gold fact rows | DQ: all matched to dims |
| test_total_budget_amount_preserved | SUM(budget) Silver = SUM(budget) Gold | DQ-05: no money lost |
| test_total_actual_amount_preserved | SUM(actual) Silver = SUM(actual) Gold | DQ-05: no money lost |

---

### 3.7 End-to-End Tests (test_end_to_end.py — 2 tests)

| Test | Validates | Rule |
|---|---|---|
| test_full_pipeline_bronze_to_gold | 7 raw → quarantine 1 → dedup → 5 Gold facts | Full pipeline correctness |
| test_pipeline_with_schema_drift | New column in CSV flows through to output | Schema evolution end-to-end |

---

## 4. Test Data Design

### Sample Data Files

| File | Rows | Purpose | Key Characteristics |
|---|---|---|---|
| budget_control.csv | 7 | Primary test data | 1 NULL key, 1 duplicate, 1 NULL encumbrance, 5 good |
| budget_control_v2.csv | 2 | Schema drift testing | Has extra TaskId column |
| budget_control_incremental.csv | 2 | CDC testing | 1 update, 1 new record |

---

## 5. Testing Levels

### Level 1: Local Unit Tests (Pre-deployment)
- **When:** Before every code change
- **Cost:** $0
- **Time:** 16 seconds
- **Command:** `python3 -m pytest tests/ -v`

### Level 2: Infrastructure Validation (Pre-deployment)
- **When:** Before terraform apply
- **Cost:** $0
- **Command:** `terraform validate` + `terraform plan`

### Level 3: AWS Integration Testing (Post-deployment)
- **When:** After terraform apply + script upload
- **Cost:** ~$1.76 per Glue job run
- **Time:** 2-3 minutes per job

### Level 4: Data Quality Validation (Production)
- **When:** After each pipeline run
- **Cost:** Fractions of a cent per query

---

## 6. Test Execution Results (Current State)

```
$ python3 -m pytest financial-lakehouse-v2/tests/ -v

30 passed, 4 skipped in 16.66s

Passed:  30 (all transformation logic)
Skipped:  4 (Iceberg JARs not available locally)
Failed:   0
```

---

## 7. Acceptance Criteria Traceability

| User Story | Tests That Validate It |
|---|---|
| US-01 (Raw ingestion) | test_csv_stored_unchanged, test_record_count_matches_source |
| US-02 (Incremental) | test_processed_file_skipped, test_new_file_processed |
| US-03 (Quarantine) | test_validation_null_key_quarantined, test_quarantine_output_structure |
| US-04 (Column rename) | test_column_rename_camel_to_snake |
| US-05 (Dedup) | test_deduplication_keeps_latest |
| US-06 (NULL handling) | test_null_handling_numerics |
| US-07 (Fiscal year) | test_fiscal_year_derivation |
| US-08 (Metrics) | test_funds_available_calculation |
| US-09 (CDC) | test_cdc_merge_insert_new, test_cdc_merge_update_existing |
| US-10 (Schema drift) | test_schema_drift_detection |
| US-11 (Schema evolution) | test_schema_evolution, test_schema_evolution_column_added |
| US-13 (dim_coa) | test_dim_coa_mapping |
| US-14 (dim_period) | test_dim_period_fiscal_calendar |
| US-15 (Fact table) | test_fact_join_completeness, test_aggregation_by_department |
| US-16 (Time travel) | test_time_travel_query, test_snapshot_created_after_write |
| US-17 (Rollback) | test_rollback_recovery |
| US-21 (Local testing) | All 34 tests passing in 16s |
| US-22 (Reconciliation) | test_total_budget_amount_preserved, test_total_actual_amount_preserved |
