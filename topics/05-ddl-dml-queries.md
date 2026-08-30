# DDL & DML Queries

## Overview
This domain tests core SQL command syntax and Snowflake-specific behavior around table types, transactions, and constraints. It's one of the more "pure SQL" domains, but with Snowflake-specific twists that the exam loves to probe.

---

## 1. DDL — Data Definition Language

DDL commands define/modify the structure of objects (databases, schemas, tables, warehouses, etc.), not the data inside them.

```sql
CREATE DATABASE my_db;
CREATE SCHEMA my_db.my_schema;
CREATE TABLE my_schema.my_table (id INT, name STRING);

ALTER TABLE my_table ADD COLUMN email STRING;
ALTER TABLE my_table RENAME COLUMN name TO full_name;

DROP TABLE my_table;
TRUNCATE TABLE my_table;   -- removes all rows, keeps table structure
```

**`DROP` vs `TRUNCATE`:**
- `DROP TABLE` removes the table entirely (structure + data). Recoverable via `UNDROP` during the Time Travel retention period.
- `TRUNCATE TABLE` removes all rows but keeps the table definition. Also generates a new table version — the emptied data is technically still recoverable via Time Travel for that retention window (this trips people up — many assume TRUNCATE is unrecoverable, but Snowflake's Time Travel still applies).

**`CREATE OR REPLACE`:**
```sql
CREATE OR REPLACE TABLE my_table (id INT);
```
- Common Snowflake pattern — atomically drops (if exists) and recreates an object in one statement. Note: the old table's data is gone (new empty table), but recoverable briefly via Time Travel like any DROP.

---

## 2. Table Types

| Type | Time Travel | Fail-safe | Use Case |
|---|---|---|---|
| **Permanent** | Up to 90 days (Enterprise+) / 1 day (Standard) | 7 days | Default table type, standard production data |
| **Transient** | Up to 1 day only | **None** | Data you don't need long-term recovery for (staging tables, ETL intermediates) — saves storage cost |
| **Temporary** | Up to 1 day only | **None** | Session-scoped only — automatically dropped when the session ends |

**Exam trap:** Transient and Temporary tables both skip Fail-safe (saving storage costs), but Temporary tables *also* disappear entirely when the session ends — Transient tables persist across sessions.

---

## 3. DML — Data Manipulation Language

```sql
INSERT INTO my_table (id, name) VALUES (1, 'Alice');

UPDATE my_table SET name = 'Bob' WHERE id = 1;

DELETE FROM my_table WHERE id = 1;

MERGE INTO target t
USING source s ON t.id = s.id
WHEN MATCHED THEN UPDATE SET t.name = s.name
WHEN NOT MATCHED THEN INSERT (id, name) VALUES (s.id, s.name);
```

- **`MERGE`** is the go-to for upsert logic (insert if new, update if existing) in a single statement — very commonly used in ETL/CDC pipelines (often paired with Streams, see Domain 3).

---

## 4. Transactions

```sql
BEGIN;
UPDATE my_table SET name = 'Bob' WHERE id = 1;
COMMIT;   -- or ROLLBACK;
```

- Snowflake auto-commits by default for individual statements outside an explicit transaction block.
- `BEGIN` / `COMMIT` / `ROLLBACK` let you group multiple statements atomically.
- DDL statements (like `CREATE`, `ALTER`) **implicitly commit** — you cannot roll back a DDL statement inside a transaction the way you can DML.

---

## 5. Constraints

- Snowflake supports standard constraint syntax (`PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `NOT NULL`) **but does not enforce most of them** — they exist mainly as metadata/documentation for query optimization and BI tool relationship discovery.
- **`NOT NULL` is the one exception that IS actually enforced.**

```sql
CREATE TABLE my_table (
  id INT PRIMARY KEY,       -- NOT enforced, informational only
  email STRING NOT NULL     -- enforced
);
```

**This is one of the single most-tested facts in this domain** — many exam questions test whether you know PK/FK/UNIQUE constraints are informational-only in Snowflake.

---

## Sample Questions & Answers

**Q1. Which constraint type does Snowflake actually enforce?**
A) PRIMARY KEY
B) FOREIGN KEY
C) UNIQUE
D) NOT NULL

**Answer: D — NOT NULL.** PRIMARY KEY, FOREIGN KEY, and UNIQUE are supported syntactically but are informational only — Snowflake does not reject data that violates them.

---

**Q2. What happens to the data in a table after `TRUNCATE TABLE my_table;`?**
A) The table and its data are permanently deleted with no recovery option
B) All rows are removed, but the table structure remains, and the removed data may still be recoverable via Time Travel
C) The table is renamed and hidden
D) TRUNCATE is not supported in Snowflake

**Answer: B.** TRUNCATE clears the data but keeps the table definition, and because Snowflake versions table data under the hood, you can typically still use Time Travel to query/recover the pre-truncate state within the retention window.

---

**Q3. Which table type does NOT have Fail-safe and is automatically dropped at the end of the session?**
A) Permanent
B) Transient
C) Temporary
D) External

**Answer: C — Temporary.** Transient tables also skip Fail-safe but persist beyond the session; Temporary tables are both Fail-safe-free AND session-scoped.

---

**Q4. Which statement best describes `CREATE OR REPLACE TABLE`?**
A) It only works if the table doesn't already exist
B) It atomically drops the existing table (if any) and creates a new one, in a single statement
C) It appends new columns to the existing table without losing data
D) It is only available for external tables

**Answer: B.** It's a convenience/atomicity pattern common in Snowflake scripts and pipelines — but be aware the old table's data is gone (replaced with an empty new table), only recoverable briefly via Time Travel like any drop.

---

**Q5. In a `MERGE` statement, what is the typical use case?**
A) Deleting duplicate rows only
B) Performing an upsert — insert new rows and update existing matching rows in a single statement
C) Creating a new table from a query
D) Renaming columns

**Answer: B.** `MERGE` is Snowflake's (and standard SQL's) mechanism for combining INSERT and UPDATE logic based on a join condition between a source and target — extremely common in incremental/CDC-style loading pipelines.

## Notes
_(Add your own notes here as you study)_

## Practice Questions Log
_(Track questions you got wrong and why)_
