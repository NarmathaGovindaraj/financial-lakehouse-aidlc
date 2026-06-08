# AIDLC Stage 4: Functional Design

## Project: COS Financial Lakehouse V2

---

## 1. Data Flow Specification

```
Oracle Fusion → BICC → OCI Object Storage → AWS S3 (Landing)
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ Job 1: COS-Oracle-to-Bronze-V2                              │
│ Input:  s3://cos-financial-lakehouse-v2/landing/{PVO}/      │
│ Output: s3://cos-financial-lakehouse-v2/bronze/{table}/     │
│         load_date=YYYY-MM-DD/{original_filename}.csv        │
│ Logic:  Copy as-is. No transformation.                      │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ Job 2: COS-Bronze-to-Silver-V2                              │
│ Input:  s3://.../bronze/budget_control/load_date=today/     │
│ Output: Iceberg → cos_silver_v2.budget_control_clean        │
│         Parquet → s3://.../silver/quarantine/               │
│ Logic:  12-step processing (see below)                      │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ Job 3: COS-Silver-to-Gold-V2                                │
│ Input:  Iceberg → cos_silver_v2.budget_control_clean        │
│ Output: Iceberg → cos_gold_v2.dim_coa                      │
│         Iceberg → cos_gold_v2.dim_period                    │
│         Iceberg → cos_gold_v2.fact_budgetary_control        │
│ Logic:  Build dimensions + fact table (see below)           │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Bronze Layer — Functional Specification

### Input
| Property | Value |
|---|---|
| Source | `s3://cos-financial-lakehouse-v2/landing/{PVO_name}/` |
| Format | CSV with header row |
| Encoding | UTF-8 |
| Delimiter | Comma |
| Schema | Oracle CamelCase (e.g., CodeCombinationId, BudgetAmount) |

### Processing Rules
| Rule | Description |
|---|---|
| R-BRZ-01 | Copy CSV file to Bronze partition without modification |
| R-BRZ-02 | Partition by `load_date=YYYY-MM-DD` |
| R-BRZ-03 | Preserve original filename |
| R-BRZ-04 | No schema validation, no type casting, no column rename |
| R-BRZ-05 | Skip if no files in landing zone |
| R-BRZ-06 | Job bookmark tracks processed dates |

### Output
| Property | Value |
|---|---|
| Location | `s3://cos-financial-lakehouse-v2/bronze/{table}/load_date=YYYY-MM-DD/` |
| Format | CSV (identical to source) |
| Partitioning | By load_date |

---

## 3. Silver Layer — Functional Specification (12 Steps)

### Input
| Property | Value |
|---|---|
| Source | `s3://cos-financial-lakehouse-v2/bronze/budget_control/load_date=today/` |
| Format | CSV with Oracle CamelCase headers |
| Expected Columns | CodeCombinationId, PeriodName, BudgetName, BudgetAmount, ActualAmount, EncumbranceAmount, CommitmentAmount, ObligationAmount, FundsAvailable, LastUpdateDate, ProjectId |

### Step-by-Step Processing

| Step | Name | Input | Output | Rule |
|---|---|---|---|---|
| 1 | Read Bronze CSV | CSV from S3 | DataFrame | Read with inferSchema=true, header=true |
| 2 | Schema Drift Detection | DataFrame columns | Log + ALTER TABLE | Compare actual columns vs EXPECTED_COLUMNS; detect new columns |
| 3 | Schema Evolution | New columns detected | Iceberg table updated | ALTER TABLE ADD COLUMNS for each new column |
| 4 | Validation | DataFrame | good_df + bad_df | Filter: key column IS NOT NULL → good; IS NULL → quarantine |
| 5 | Quarantine Write | bad_df | Parquet in quarantine/ | Write with quarantine_reason + quarantine_date |
| 6 | Column Rename | good_df (CamelCase) | good_df (snake_case) | CamelCase → snake_case using regex |
| 7 | Ingestion Metadata | DataFrame | + 3 columns | Add bicc_extract_id, ingestion_timestamp, load_date |
| 8 | Cleanse | DataFrame | cleaned DataFrame | Trim strings, uppercase period_name, COALESCE NULLs → 0.0, filter invalid period format |
| 9 | Dedup | DataFrame | deduped DataFrame | ROW_NUMBER OVER (PARTITION BY ccid, period ORDER BY last_update_date DESC), keep rn=1 |
| 10 | Fiscal Year | DataFrame | + fiscal_year column | JUL-DEC → year+1, JAN-JUN → year |
| 11 | Metrics | DataFrame | + derived columns | expenditure=actual, funds_available=budget-actual-encumbrance, burn_rate=(actual+encumbrance)/budget |
| 12 | MERGE INTO | Final DataFrame | Iceberg table updated | MERGE: matched+newer → UPDATE, not matched → INSERT |

### Transformation Rules Detail

**R-SLV-01: Validation**
```
Good: CodeCombinationId IS NOT NULL
Bad:  CodeCombinationId IS NULL → quarantine with reason "NULL_KEY_COLUMN"
```

**R-SLV-02: Column Mapping**
| Source (Oracle) | Target (Silver) |
|---|---|
| CodeCombinationId | code_combination_id |
| PeriodName | period_name |
| BudgetName | budget_name |
| BudgetAmount | budget_amount |
| ActualAmount | actual_amount |
| EncumbranceAmount | encumbrance_amount |
| CommitmentAmount | commitment_amount |
| ObligationAmount | obligation_amount |
| FundsAvailable | funds_available |
| LastUpdateDate | last_update_date |
| ProjectId | project_id |

**R-SLV-03: NULL Handling**
| Column | Rule |
|---|---|
| actual_amount | COALESCE(value, 0.0) |
| encumbrance_amount | COALESCE(value, 0.0) |
| commitment_amount | COALESCE(value, 0.0) |
| obligation_amount | COALESCE(value, 0.0) |

**R-SLV-04: Deduplication**
```
Key:     code_combination_id + period_name
Order:   last_update_date DESC
Keep:    Row number = 1 (latest)
```

**R-SLV-05: Fiscal Year**
```
IF month IN (JUL, AUG, SEP, OCT, NOV, DEC) THEN year + 1
ELSE year
```

**R-SLV-06: Metrics**
```
expenditure_amount = actual_amount
funds_available = budget_amount - actual_amount - encumbrance_amount
burn_rate = ROUND((actual_amount + encumbrance_amount) / budget_amount, 4)
  IF budget_amount = 0 THEN burn_rate = 0.0
```

**R-SLV-07: MERGE Logic**
```sql
MERGE INTO silver_table AS target
USING source AS source
ON target.code_combination_id = source.code_combination_id
   AND target.period_name = source.period_name
WHEN MATCHED AND source.last_update_date > target.last_update_date THEN
    UPDATE SET all non-key columns
WHEN NOT MATCHED THEN
    INSERT all columns
```

### Output Schema — cos_silver_v2.budget_control_clean

| Column | Type | Nullable | Source |
|---|---|---|---|
| code_combination_id | BIGINT | No | Oracle: CodeCombinationId |
| period_name | STRING | No | Oracle: PeriodName (trimmed, uppercased) |
| budget_name | STRING | Yes | Oracle: BudgetName |
| budget_amount | DOUBLE | No | Oracle: BudgetAmount |
| actual_amount | DOUBLE | No | Oracle: ActualAmount (NULL→0.0) |
| encumbrance_amount | DOUBLE | No | Oracle: EncumbranceAmount (NULL→0.0) |
| commitment_amount | DOUBLE | No | Oracle: CommitmentAmount (NULL→0.0) |
| obligation_amount | DOUBLE | No | Oracle: ObligationAmount (NULL→0.0) |
| expenditure_amount | DOUBLE | No | Derived: = actual_amount |
| funds_available | DOUBLE | No | Derived: budget - actual - encumbrance |
| burn_rate | DOUBLE | No | Derived: (actual + encumbrance) / budget |
| last_update_date | TIMESTAMP | Yes | Oracle: LastUpdateDate |
| fiscal_year | INT | No | Derived from period_name |
| bicc_extract_id | STRING | No | Generated: BICC_YYYYMMDD |
| ingestion_timestamp | TIMESTAMP | No | Generated: job execution time |
| load_date | STRING | No | Bronze partition date |
| dw_insert_date | TIMESTAMP | No | Generated: current_timestamp() |
| dw_update_date | TIMESTAMP | No | Generated: current_timestamp() |

---

## 4. Gold Layer — Functional Specification

### dim_coa (Chart of Accounts)

| Column | Type | Source | Description |
|---|---|---|---|
| coa_key | BIGINT | Generated | Surrogate key |
| code_combination_id | BIGINT | Silver | Natural key |
| fund | STRING | Hardcoded/COA PVO | Fund code |
| fund_description | STRING | Hardcoded/COA PVO | Fund name |
| department | STRING | Derived | Department code |
| department_description | STRING | Mapped | Department name |
| account | STRING | Hardcoded/COA PVO | Account code |
| account_description | STRING | Hardcoded/COA PVO | Account name |
| account_type | STRING | Hardcoded/COA PVO | E=Expense, R=Revenue |
| is_current | BOOLEAN | SCD flag | True = active |
| dw_load_date | TIMESTAMP | Generated | Load timestamp |

### dim_period (Fiscal Calendar)

| Column | Type | Source | Description |
|---|---|---|---|
| period_key | BIGINT | Generated | Surrogate key |
| period_name | STRING | Silver | e.g., "JUL-2024" |
| fiscal_year | INT | Silver | e.g., 2025 |
| fiscal_quarter | INT | Derived | Q1=Jul-Sep, Q2=Oct-Dec, Q3=Jan-Mar, Q4=Apr-Jun |
| fiscal_month | INT | Derived | 1=Jul, 2=Aug, ..., 12=Jun |
| calendar_year | INT | Derived | e.g., 2024 |
| calendar_month_name | STRING | Derived | e.g., "JUL" |
| dw_load_date | TIMESTAMP | Generated | Load timestamp |

### fact_budgetary_control

| Column | Type | Source | Description |
|---|---|---|---|
| bc_key | BIGINT | Generated | Surrogate key |
| coa_key | BIGINT | dim_coa join | FK to dim_coa |
| period_key | BIGINT | dim_period join | FK to dim_period |
| code_combination_id | BIGINT | Silver | Degenerate dimension |
| budget_amount | DOUBLE | Silver | Budget |
| actual_amount | DOUBLE | Silver | Actual spend |
| encumbrance_amount | DOUBLE | Silver | Encumbered |
| commitment_amount | DOUBLE | Silver | Committed |
| obligation_amount | DOUBLE | Silver | Obligated |
| expenditure_amount | DOUBLE | Silver | Expenditure |
| funds_available | DOUBLE | Silver | Remaining funds |
| burn_rate | DOUBLE | Silver | Utilization % |
| fiscal_year | INT | Silver | Partition key |
| dw_load_date | TIMESTAMP | Generated | Load timestamp |

---

## 5. Data Quality Rules Summary

| ID | Rule | Layer | Action on Violation |
|---|---|---|---|
| DQ-01 | code_combination_id NOT NULL | Silver | Quarantine |
| DQ-02 | period_name matches ^[A-Z]{3}-\d{4}$ | Silver | Remove from processing |
| DQ-03 | No duplicate (ccid + period) | Silver | Keep latest, discard older |
| DQ-04 | Numeric NULLs | Silver | Replace with 0.0 |
| DQ-05 | SUM(budget) Silver = SUM(budget) Gold | Gold | Alert if mismatch |
| DQ-06 | All fact rows match dimensions | Gold | Exclude unmatched from fact |

---

## 6. Reconciliation Checkpoints

| Checkpoint | Expected Relationship | Validation Query |
|---|---|---|
| Bronze vs Source | Bronze rows = Source CSV rows | Compare file row count |
| Bronze vs Silver | Bronze ≥ Silver (quarantine + dedup) | COUNT comparison |
| Silver vs Gold | Silver = Gold fact (all matched) | COUNT comparison |
| Budget totals | SUM(Silver.budget) = SUM(Gold.budget) | SUM comparison |
| Actual totals | SUM(Silver.actual) = SUM(Gold.actual) | SUM comparison |
