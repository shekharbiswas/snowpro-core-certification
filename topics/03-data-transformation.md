# Data Transformation

## Overview
This domain covers transforming data during and after load — working with semi-structured data, using Streams/Tasks for pipelines, and views. Expect several questions on VARIANT/FLATTEN syntax and Streams+Tasks mechanics.



## 1. Transforming Data During Load

`COPY INTO` supports a **SELECT-based transformation** at load time (instead of loading raw columns 1:1):

```sql
COPY INTO my_table (col1, col2, col3)
FROM (
  SELECT $1, $2, UPPER($3)
  FROM @my_stage/data.csv
)
FILE_FORMAT = (TYPE = 'CSV');
```

- `$1`, `$2`, etc. reference columns **by position** in the source file.
- You can apply functions, reorder columns, skip columns, or do simple casts right in the COPY statement.
- **Limitation:** only simple, row-level transformations are supported here — no joins, aggregations, or window functions during a COPY INTO.



## 2. Semi-Structured Data: VARIANT & FLATTEN

Semi-structured formats (JSON, Avro, Parquet, ORC, XML) load into a single **VARIANT** column, which stores the whole document per row.

### Querying VARIANT data
```sql
SELECT raw_data:name::STRING AS name,
       raw_data:age::INT AS age
FROM my_table;
```
- `:` = accessing a key inside the VARIANT.
- `::` = explicit cast to a native type (VARIANT is untyped until cast).

### FLATTEN — unnesting arrays/objects into rows
```sql
SELECT value:item::STRING AS item_name
FROM my_table, LATERAL FLATTEN(input => raw_data:items);
```
- `FLATTEN` is a table function that turns a nested array or object into multiple rows.
- Almost always used with `LATERAL` so it can reference the outer table's column.

**Why this matters for the exam:** Snowflake's big selling point for semi-structured data is that you *don't* need to define a rigid schema up front — you load the raw JSON as VARIANT, then flatten/cast as needed at query time.



## 3. Streams — Change Data Capture (CDC)

A **Stream** tracks changes (inserts, updates, deletes) made to a table since the last time the stream was consumed.

```sql
CREATE STREAM my_stream ON TABLE my_table;

SELECT * FROM my_stream;  -- shows changed rows + metadata columns
```

Metadata columns added by a stream:
- `METADATA$ACTION` → INSERT or DELETE (an UPDATE shows as a DELETE + INSERT pair)
- `METADATA$ISUPDATE` → TRUE/FALSE
- `METADATA$ROW_ID`

- A stream has an **offset** — it only shows changes since that offset. Once you `SELECT`/consume it inside a transaction (e.g. via a Task using `INSERT ... SELECT FROM stream`), the offset advances.
- Streams don't store the actual changed data — they use Time Travel metadata under the hood to compute the delta.


## 4. Tasks — Scheduling SQL

A **Task** runs a single SQL statement (or calls a stored procedure) on a schedule or when triggered.

```sql
CREATE TASK my_task
  WAREHOUSE = my_wh
  SCHEDULE = '5 MINUTE'
AS
  INSERT INTO target_table
  SELECT * FROM my_stream;
```

- Tasks can be **chained** (one task triggers the next) to build simple pipelines.
- Common pattern: **Stream + Task** = a lightweight, incremental ETL pipeline (task runs periodically, consumes only new changes via the stream).
- A task only actually runs its SQL if there's data in the stream when combined with `WHEN SYSTEM$STREAM_HAS_DATA('my_stream')`.



## 5. Views & Materialized Views

| | View | Materialized View |
|---|---|---|
| Storage | No data stored, just a saved query | Stores actual precomputed results |
| Refresh | Always runs live query | Auto-refreshed by Snowflake in background |
| Cost | Compute cost only when queried | Storage cost + background maintenance compute |
| Availability | All editions | Enterprise edition and above |
| Best for | Simplifying/reusing queries | Speeding up expensive, frequently-run aggregations on data that doesn't change too often |



## 6. UDFs & Stored Procedures (lighter exam weight, but know the basics)
- **UDF** (User-Defined Function): returns a value, used inline in SQL (`SELECT my_udf(col) FROM t`). Written in SQL, JavaScript, Python, Java, etc.
- **Stored Procedure**: performs actions/logic, can contain control flow (loops, conditionals), called with `CALL my_proc()`. Does not have to return a value used inline in a query the way a UDF does.

---

## Sample Questions & Answers

**Q1. Which command is used to unnest a VARIANT array into multiple rows?**
A) `UNNEST()`
B) `FLATTEN()`
C) `EXPLODE()`
D) `PARSE_JSON()`

**Answer: B — `FLATTEN()`.** It's a table function typically used with `LATERAL` to expand array/object elements into separate rows. (`PARSE_JSON` just converts a string into a VARIANT — it doesn't unnest anything.)



**Q2. A Stream on a table shows a row was updated. How does METADATA$ACTION represent this?**
A) A single row with METADATA$ACTION = 'UPDATE'
B) Two rows: one DELETE and one INSERT, with METADATA$ISUPDATE = TRUE
C) No row appears until the transaction commits twice
D) METADATA$ACTION = 'MODIFY'

**Answer: B.** Snowflake Streams represent an UPDATE as a paired DELETE + INSERT, both flagged with `METADATA$ISUPDATE = TRUE` so you can tell it apart from an unrelated delete followed by an unrelated insert.



**Q3. What is the main difference between a standard View and a Materialized View?**
A) Views can only be created by ACCOUNTADMIN
B) Materialized Views physically store query results and are refreshed automatically; standard Views run the underlying query each time
C) Views are faster because they cache data automatically
D) Materialized Views are available on all Snowflake editions

**Answer: B.** Materialized Views trade storage + maintenance cost for faster reads on expensive/frequent queries. They also require Enterprise edition or higher — not all editions (ruling out D).



**Q4. Which of the following is true about combining Streams and Tasks?**
A) A Task automatically deletes the Stream after running
B) Streams and Tasks cannot be used together
C) A Task can check `SYSTEM$STREAM_HAS_DATA()` to only run when the stream has new changes
D) Streams can only be consumed by Snowpipe, not Tasks

**Answer: C.** This Stream + Task pattern is the standard way to build simple, cost-efficient incremental pipelines in Snowflake — the task avoids running (and burning compute) when there's nothing new to process.


**Q5. Can you perform a JOIN inside the SELECT of a COPY INTO transformation?**
A) Yes, any SQL is supported
B) No — only simple row-level expressions (column selection, casts, functions) are supported, no joins/aggregations
C) Only if using an external stage
D) Only when loading Parquet files

**Answer: B.** COPY INTO transformations are intentionally limited to simple per-row transformations during the load itself — more complex transformation logic (joins, aggregations) should happen after loading, typically via Streams/Tasks, stored procedures, or downstream `INSERT INTO ... SELECT` statements.

## Notes
_(Add your own notes here as you study)_

## Practice Questions Log
_(Track questions you got wrong and why)_
