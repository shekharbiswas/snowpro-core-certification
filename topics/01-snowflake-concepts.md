# Snowflake Concepts & Architecture

## Architecture Overview
Snowflake uses a **multi-cluster shared data architecture** with three independently scalable layers:

1. **Database Storage Layer** — Data is stored in Snowflake's internal format (compressed, columnar, micro-partitions) in cloud storage (S3/Blob/GCS depending on cloud provider). Not directly accessible by users.
2. **Query Processing (Compute) Layer** — Virtual Warehouses. Each is an independent MPP compute cluster. Warehouses don't share compute resources with each other, so one heavy workload doesn't slow another.
3. **Cloud Services Layer** — Manages authentication, infrastructure, metadata, query parsing/optimization, access control. Runs on Snowflake-managed compute (not customer virtual warehouses).

## Micro-Partitions
- Data is automatically divided into micro-partitions (50–500 MB uncompressed each).
- Columnar storage within each micro-partition.
- Metadata (min/max values, distinct counts) is stored per column per micro-partition — enables **pruning** (skipping irrelevant partitions during queries) without manual indexing.

## Editions
| Edition | Key Features |
|---|---|
| Standard | Core features, standard Time Travel (1 day) |
| Enterprise | + longer Time Travel (up to 90 days), multi-cluster warehouses, materialized views |
| Business Critical | + HIPAA/PCI support, customer-managed keys (Tri-Secret Secure), failover/replication |
| Virtual Private Snowflake (VPS) | Dedicated, fully isolated Snowflake environment |

## Supported Clouds
AWS, Microsoft Azure, Google Cloud Platform — Snowflake runs natively on each; cross-cloud/cross-region replication is possible (Business Critical+).

## Key Terms to Know
- **Account** — top-level container tied to a region/cloud platform.
- **Organization** — groups multiple Snowflake accounts under one entity.
- **Database → Schema → Table/View** hierarchy.
- **Compute is billed per-second (min 60 sec)** based on warehouse size; storage billed separately based on average monthly usage.

## Common Exam Traps
- Storage and compute billing are **separate** — you pay for storage even if no warehouse is running.
- Cloud Services layer usage is only billed if it exceeds 10% of daily compute credit usage.
- Snowflake has **no indexes** in the traditional sense — pruning via micro-partition metadata replaces that need.

## Notes
_(Add your own notes here as you study)_

## Practice Questions Log
_(Track questions you got wrong and why)_
