# Data Loading

## Overview
This domain tests how well you understand getting data **into** Snowflake — both in bulk (batch) and continuously (streaming-style). About 10-15% of the exam touches loading concepts, so know the mechanics of stages, file formats, and COPY INTO cold.

---

## 1. Stages — Where Files Land Before Loading

A **stage** is just a pointer to a storage location that Snowflake can read files from.

| Stage Type | Description |
|---|---|
| **User Stage** (`@~`) | Automatically created per user. Only that user can use it. Not sharable. Good for one-off personal loads. |
| **Table Stage** (`@%tablename`) | Automatically created per table. Files here are tied to loading into that ONE table only. |
| **Internal Named Stage** | Created explicitly (`CREATE STAGE my_stage`). Storage is managed by Snowflake. Reusable across users/tables, can set file format defaults on it. |
| **External Stage** | Points to a location YOU own — an S3 bucket, Azure Blob container, or GCS bucket. Requires storage integration or credentials. |

**Exam tip:** Internal stages are Snowflake-managed storage; External stages are customer-owned cloud storage that Snowflake reads from/writes to.

### Loading files into an internal stage
```sql
PUT file:///local/path/data.csv @my_stage;
```
`PUT` only works for internal stages (you're uploading local files into Snowflake-managed storage). For external stages, you upload directly via your cloud provider's tools — Snowflake just reads from there.

---

## 2. File Formats

Snowflake supports: **CSV/delimited, JSON, Avro, ORC, Parquet, XML**.

- Create a reusable file format object so you're not repeating options every load:
```sql
CREATE FILE FORMAT my_csv_format
  TYPE = 'CSV'
  FIELD_DELIMITER = ','
  SKIP_HEADER = 1
  NULL_IF = ('NULL', 'null');
```
- Semi-structured formats (JSON, Avro, Parquet, ORC) get loaded into a single **VARIANT** column by default unless you explicitly map fields to columns.

---

## 3. COPY INTO — The Core Loading Command

```sql
COPY INTO my_table
  FROM @my_stage
  FILE_FORMAT = (FORMAT_NAME = 'my_csv_format')
  ON_ERROR = 'CONTINUE';
```

Key options to memorize:
- **`ON_ERROR`**: `ABORT_STATEMENT` (default — stops on first error), `CONTINUE` (skips bad rows, loads the rest), `SKIP_FILE`, `SKIP_FILE_<n>`.
- **`FORCE = TRUE`**: reloads files even if already loaded before (normally Snowflake tracks load history for 64 days and won't reload the same file twice — this is a safety feature to prevent duplicate loads).
- **`PURGE = TRUE`**: deletes files from the stage after a successful load.
- **`VALIDATION_MODE`**: lets you test a load (return errors) without actually inserting data.

**Why "won't reload the same file" matters for the exam:** Snowflake uses file name + file hash (checksum) to track what's already been loaded, avoiding accidental duplicate loads if you re-run the same COPY INTO.

---

## 4. Snowpipe — Continuous Loading

Snowpipe loads data automatically and incrementally as soon as new files land in a stage — no manual COPY INTO needed.

- Triggered by **cloud provider event notifications** (e.g., S3 event notification → SQS → Snowpipe) or by calling the **REST API** manually.
- Uses **serverless compute** (Snowflake-managed, not your virtual warehouse) — billed per second of compute used for the pipe, separate from warehouse credits.
- Good for near-real-time ingestion (files arriving continuously from an application, IoT, logs, etc.)
- Defined with `CREATE PIPE`:
```sql
CREATE PIPE my_pipe AS
  COPY INTO my_table
  FROM @my_stage
  FILE_FORMAT = (FORMAT_NAME = 'my_csv_format');
```

**Exam distinction:** Snowpipe = automated, event-driven, serverless, micro-batches. Regular `COPY INTO` = manual/scheduled, run on your own virtual warehouse.

---

## 5. Bulk Loading vs Snowpipe — When to Use Which

| | Bulk COPY INTO | Snowpipe |
|---|---|---|
| Trigger | Manual or scheduled (e.g., via Task) | Automatic (event-based) |
| Compute | Your virtual warehouse | Snowflake-managed serverless |
| Best for | Large batch loads, historical backfills | Continuous, smaller/frequent file arrivals |
| Latency | Depends on your schedule | Near real-time (seconds to minutes) |

---

## Common Exam Traps
- `PUT` command is client-side (uploads local file to internal stage) — it does **not** load data into a table by itself. You still need `COPY INTO` afterward.
- Table stages **cannot** be used to load into a different table than the one they're attached to.
- Snowpipe does NOT guarantee exactly-once loading in extreme edge cases, but it avoids duplicate loads under normal operation via the same file-tracking metadata as COPY INTO.
- Load metadata (history of loaded files) is retained for 64 days by default.

## Notes
_(Add your own notes here as you study)_

## Practice Questions Log
_(Track questions you got wrong and why)_
