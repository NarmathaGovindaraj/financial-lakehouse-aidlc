# AIDLC Stage 3: Architecture Design

## Project: COS Financial Lakehouse V2

---

## Architecture Decision Records (ADRs)

### ADR-1: Storage — Amazon S3

| Aspect | Detail |
|---|---|
| Decision | Use S3 as the sole storage layer for all lakehouse data |
| Why | Cheapest, most durable (11 9s), infinite scale, AWS-native integration |
| Alternatives rejected | EFS/EBS (not analytics-suitable), Redshift Managed Storage (vendor lock), HDFS (not serverless) |
| Trade-offs | No built-in update/delete (resolved by Iceberg); eventually consistent for overwrites |
| Risks | Linear cost growth with volume; single-region without replication |
| Scalability | Infinite; S3 Intelligent-Tiering and Glacier for cost optimization at scale |

---

### ADR-2: Table Format — Apache Iceberg

| Aspect | Detail |
|---|---|
| Decision | Use Iceberg for Silver and Gold layer tables |
| Why | Only open format natively supported by both Athena AND Glue; MERGE INTO, time travel, schema evolution |
| Alternatives rejected | Delta Lake (Databricks-centric, not Athena-native), Hudi (complex config), Plain Parquet (no MERGE/time travel) |
| Trade-offs | Slightly more storage (metadata); requires Glue 4.0+; snapshot maintenance needed |
| Risks | Specific --conf and --datalake-formats config required; metadata growth at extreme scale |
| Scalability | Hidden partitioning, partition evolution, compaction; multi-engine support (Glue/EMR/Spark) |

---

### ADR-3: Compute — AWS Glue 4.0 (PySpark)

| Aspect | Detail |
|---|---|
| Decision | Use Glue 4.0 serverless Spark for all ETL processing |
| Why | Serverless (no idle cost), native Iceberg support, pay-per-run, Glue Catalog integration |
| Alternatives rejected | EMR (cluster management), EMR Serverless (less docs), Lambda (no Spark), Databricks (vendor lock) |
| Trade-offs | ~60s cold start; $0.44/DPU-hour; limited Spark config control vs self-managed |
| Risks | Debugging harder (CloudWatch logs only); version upgrades can break config |
| Scalability | Scale workers from 2 to 10+; upgrade to G.2X; migrate to EMR Serverless if cost threshold exceeded |

---

### ADR-4: Query Engine — Amazon Athena

| Aspect | Detail |
|---|---|
| Decision | Use Athena as the SQL query engine for all analytics queries |
| Why | Serverless, $5/TB scanned, native Iceberg support, Power BI ODBC compatible |
| Alternatives rejected | Redshift ($200+/month minimum), Presto on EMR (cluster needed), direct S3 from Power BI (can't read Iceberg) |
| Trade-offs | No caching; concurrent query limits; cost spikes on unfiltered full-table scans |
| Risks | Cost governance needed; Athena v3 required for Iceberg; workgroup configuration critical |
| Scalability | Auto-scales; add per-query data scan limits; add Redshift Serverless if sub-second latency needed |

---

### ADR-5: Catalog — AWS Glue Data Catalog

| Aspect | Detail |
|---|---|
| Decision | Use Glue Data Catalog as the unified metadata store |
| Why | Free, shared by Glue ETL + Athena queries, native Iceberg registration |
| Alternatives rejected | Hive Metastore on EMR (requires running cluster), Lake Formation (complex for Phase 1) |
| Trade-offs | API rate limits; no column-level security without Lake Formation |
| Risks | Lake Formation interference if enabled account-wide; regional (no multi-region) |
| Scalability | Supports thousands of tables; add Lake Formation for fine-grained access later |

---

### ADR-6: Orchestration — Glue Workflows + Triggers

| Aspect | Detail |
|---|---|
| Decision | Use Glue Workflows with scheduled + conditional triggers |
| Why | Native, free, sufficient for linear 3-job pipeline |
| Alternatives rejected | MWAA/Airflow ($300+/month), Step Functions (overkill for linear chain), EventBridge+Lambda (extra components) |
| Trade-offs | Linear chains only — no parallel branches or complex DAGs; limited monitoring |
| Risks | Won't scale beyond 5-6 jobs; no built-in failure notification; no retry with backoff |
| Scalability | Migrate to Step Functions or MWAA when complexity increases; Glue jobs unchanged |

---

### ADR-7: Infrastructure as Code — Terraform

| Aspect | Detail |
|---|---|
| Decision | Use Terraform for all AWS resource provisioning |
| Why | Mature, multi-cloud capable, plan/apply workflow, team familiarity |
| Alternatives rejected | CDK (AWS-specific, newer), CloudFormation (verbose, no plan preview), Pulumi (smaller community) |
| Trade-offs | HCL learning curve; state file management needed; doesn't handle Iceberg DDL |
| Risks | Local state file loss; provider version breaking changes; drift without explicit plan |
| Scalability | Add S3 backend + DynamoDB locking; modularize; separate workspaces; CI/CD pipeline |

---

### ADR-8: Data Architecture — Medallion (Bronze/Silver/Gold)

| Aspect | Detail |
|---|---|
| Decision | Three-layer medallion architecture: Bronze (raw CSV) → Silver (Iceberg + all processing) → Gold (star schema) |
| Why | Industry standard; clean separation of concerns; easy to debug, extend, and explain |
| Alternatives rejected | Lambda/Kappa (no streaming requirement), single-layer (no audit trail), 4-layer with Landing (unnecessary complexity) |
| Trade-offs | 3 jobs = 3x cost per run; data latency across 3 stages; Bronze duplicates source data |
| Risks | Mid-pipeline failure leaves layers inconsistent until re-run; Bronze growth over years |
| Scalability | Each new PVO adds one Bronze path + one Silver table + one Gold dimension/fact; parallelize per-PVO |

---

## Technology Stack Summary

```
┌──────────────────────────────────────────────────┐
│              COS Financial Lakehouse V2            │
├──────────────────────────────────────────────────┤
│ Storage:        Amazon S3                         │
│ Table Format:   Apache Iceberg                    │
│ Compute:        AWS Glue 4.0 (PySpark)            │
│ Query:          Amazon Athena                     │
│ Catalog:        AWS Glue Data Catalog             │
│ Orchestration:  Glue Workflows + Triggers         │
│ IaC:            Terraform                         │
│ Visualization:  Power BI (via Athena ODBC)        │
│ Pattern:        Medallion (Bronze/Silver/Gold)    │
│ Region:         us-east-1                         │
└──────────────────────────────────────────────────┘
```
