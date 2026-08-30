# SnowPro Core — Practice Exam 2

**Total Questions:** 35
**Question Types:** Multiple Choice (select 1), Multiple Select (select 2+, as noted)
**Time Limit:** 65 minutes
**Passing Guidance:** Aim for 75%+ before sitting the actual exam

---

**Instructions:** Answer all questions before checking the answer key at the bottom. Questions marked **(Multiple Select)** have more than one correct answer — select all that apply. All others are single-answer Multiple Choice.

---

**1.** In Snowflake's architecture, where is customer data physically stored?
A) On the virtual warehouse's local disk permanently
B) In cloud storage (S3/Blob/GCS) managed by the Database Storage layer, in Snowflake's internal format
C) In the Cloud Services layer
D) On the client machine running the query

**2. (Multiple Select)** Which of the following are true about Snowflake editions? (Select 2)
A) Business Critical adds support for customer-managed encryption keys (Tri-Secret Secure)
B) Standard edition includes the longest Time Travel retention by default
C) Enterprise edition adds multi-cluster warehouses and materialized views
D) VPS (Virtual Private Snowflake) is the cheapest edition

**3.** What is the main difference between a user stage and a table stage?
A) User stages can be shared across users; table stages cannot
B) A user stage is tied to a specific user; a table stage is tied to loading into one specific table only
C) Table stages support external cloud storage; user stages do not
D) There is no functional difference

**4.** Which file format types can Snowflake load natively? (choose the most complete answer)
A) CSV and JSON only
B) CSV, JSON, Avro, ORC, Parquet, XML
C) Only structured (tabular) formats
D) Only Parquet and ORC

**5.** What does the `FORCE = TRUE` option do in a COPY INTO statement?
A) Forces the load to skip all validation
B) Reloads files even if they were already loaded before (bypassing Snowflake's duplicate-load tracking)
C) Forces the warehouse to scale up during the load
D) Forces the target table to be recreated

**6.** How long does Snowflake retain load history metadata (used to prevent duplicate file loads) by default?
A) 24 hours
B) 7 days
C) 64 days
D) Indefinitely

**7. (Multiple Select)** Which of the following are advantages of Snowpipe over standard bulk COPY INTO? (Select 2)
A) Uses your own virtual warehouse, giving more control over compute size
B) Automatically triggered by new file arrival, reducing manual scheduling
C) Uses serverless compute, so no warehouse needs to be managed for the load
D) Guarantees zero latency between file arrival and query availability

**8.** What is the correct syntax pattern to unnest a nested array column called `items` inside a VARIANT column `raw_data`?
A) `SELECT items FROM my_table;`
B) `SELECT value FROM my_table, LATERAL FLATTEN(input => raw_data:items);`
C) `SELECT UNNEST(raw_data:items) FROM my_table;`
D) `SELECT raw_data.items[*] FROM my_table;`

**9.** What is the relationship between Streams and Tasks in a typical incremental pipeline pattern?
A) Tasks create Streams automatically
B) A Task periodically runs SQL that consumes new changes tracked by a Stream, often gated by SYSTEM$STREAM_HAS_DATA()
C) Streams replace the need for Tasks entirely
D) Streams and Tasks cannot be used in the same pipeline

**10.** Which edition is required to use Materialized Views?
A) Standard
B) Enterprise or higher
C) Any edition
D) Business Critical only

**11.** What does scaling OUT (adding more clusters) primarily help with, as opposed to scaling UP?
A) Making a single complex query run faster
B) Handling more concurrent queries/users without queuing
C) Reducing storage costs
D) Improving data loading speed

**12.** Which of the following correctly describes the Warehouse (local disk/SSD) cache?
A) It persists even after the warehouse is suspended
B) It stores raw data pages on the warehouse's local SSD and is lost when the warehouse suspends
C) It lives in the Cloud Services layer
D) It caches full query results for 24 hours

**13.** A resource monitor is configured with `TRIGGERS ON 100 PERCENT DO SUSPEND`. What happens at 100% credit usage?
A) All running queries are immediately cancelled
B) New queries are blocked, but currently running queries are allowed to finish
C) Nothing — SUSPEND is only a notification
D) The account is permanently locked

**14.** Which of the following is true about Snowflake constraints (PRIMARY KEY, FOREIGN KEY, UNIQUE, NOT NULL)?
A) All four are strictly enforced
B) None of them are enforced
C) Only NOT NULL is enforced; the others are informational/metadata only
D) Only PRIMARY KEY is enforced

**15.** What is the effect of `CREATE OR REPLACE TABLE` on an existing table with the same name?
A) It adds new columns without affecting existing data
B) It atomically drops the existing table and creates a new, empty one with the given definition
C) It fails with an error if the table already exists
D) It renames the old table automatically as a backup

**16. (Multiple Select)** Which table types are capped at a maximum of 1 day of Time Travel regardless of edition? (Select 2)
A) Permanent
B) Transient
C) Temporary
D) External

**17.** What is a Reader Account used for?
A) Giving a consumer without their own Snowflake account access to shared data, managed and paid for by the provider
B) A read-only replica of the provider's entire account
C) An account type used only for the Marketplace
D) A special role for read-only SQL users within the same account

**18.** Which of the following is required to share data between accounts in different cloud regions (e.g., AWS to Azure)?
A) Nothing extra — standard sharing works across any region/cloud
B) Database replication must be set up first, requiring Business Critical edition or higher
C) The consumer must create a Reader Account
D) Cross-cloud sharing is never possible

**19.** What access level does a consumer have on data received via a Share, by default?
A) Full read/write access
B) Read-only (SELECT) access
C) DDL access to alter the shared objects
D) No access until they create their own copy

**20.** What is the key difference between the Snowflake Marketplace and a private Data Exchange?
A) They use entirely different underlying sharing technology
B) Marketplace is publicly discoverable; Data Exchange is private/invite-only for a defined group
C) Only Data Exchange uses live (non-copied) data
D) Marketplace requires Business Critical edition

**21.** Which system-defined role is primarily responsible for managing users and roles (a subset of SECURITYADMIN's scope)?
A) SYSADMIN
B) USERADMIN
C) PUBLIC
D) ORGADMIN

**22.** In Snowflake's RBAC model, how are privileges typically assigned?
A) Directly to individual users
B) To roles, which are then granted to users (and roles can be granted to other roles)
C) To warehouses, which cascade to users
D) Privileges cannot be customized — only system roles exist

**23.** Every Snowflake user automatically has which default role, which carries minimal privileges?
A) SYSADMIN
B) ACCOUNTADMIN
C) PUBLIC
D) GUEST

**24.** What happens to a zero-copy clone's storage cost as the clone and source table diverge over time (new data added to each independently)?
A) Storage cost stays at zero forever
B) Storage cost increases only for the newly diverged/changed micro-partitions in each
C) The entire clone is duplicated in storage immediately upon divergence
D) The source table is charged double

**25.** What is the correct syntax to query a table as it existed at a specific past timestamp?
A) `SELECT * FROM my_table SNAPSHOT '2026-08-01';`
B) `SELECT * FROM my_table AT (TIMESTAMP => '2026-08-01 10:00:00'::TIMESTAMP);`
C) `SELECT * FROM my_table HISTORY (TIMESTAMP => '2026-08-01');`
D) `RESTORE TABLE my_table TO '2026-08-01';`

**26.** Which command recovers a dropped table within its Time Travel retention window?
A) RESTORE TABLE
B) UNDROP TABLE
C) RECOVER TABLE
D) UNDELETE TABLE

**27.** Can a user with ACCOUNTADMIN role recover data during the Fail-safe period using SQL?
A) Yes, ACCOUNTADMIN has unrestricted access to Fail-safe
B) No — Fail-safe recovery requires contacting Snowflake Support; no role can self-service it via SQL
C) Yes, but only using UNDROP
D) Yes, but only within the first 24 hours of Fail-safe

**28.** Which authentication method is most commonly used for programmatic/service accounts (e.g., CI/CD pipelines, drivers)?
A) Username/password only
B) Key-pair authentication
C) SSO exclusively
D) MFA is required for all programmatic access

**29.** What is the correct top-down object containment hierarchy in Snowflake?
A) Account → Schema → Database → Table
B) Organization → Account → Database → Schema → Table
C) Database → Organization → Account → Table
D) Table → Schema → Database → Organization

**30. (Multiple Select)** Which of the following statements about Fail-safe are true? (Select 2)
A) It is a fixed 7-day period, non-configurable
B) It applies to Transient and Temporary tables as well as Permanent
C) It applies only to Permanent tables
D) Recovery during Fail-safe requires contacting Snowflake Support

**31.** A query is re-run identically minutes later with no underlying data changes. Which cache serves the result without needing an active warehouse?
A) Local disk cache
B) Metadata cache
C) Result cache
D) None — every query always re-executes fully

**32.** What is the primary purpose of the Query Profile tool?
A) To manage user roles and grants
B) To visually diagnose query execution — showing time per operator, bytes scanned, pruning stats, and spillage
C) To configure Time Travel retention
D) To create Shares

**33.** Which of the following is TRUE regarding DML transactions in Snowflake?
A) DDL statements can be rolled back like DML within a transaction
B) BEGIN/COMMIT/ROLLBACK can group multiple DML statements atomically; DDL implicitly commits
C) Transactions are not supported in Snowflake
D) Only SELECT statements can be part of a transaction

**34.** Which of these is an accurate statement about clustering keys?
A) They are required on every table by default
B) They are most beneficial on very large tables with a well-defined, frequently filtered column
C) They primarily speed up data loading, not querying
D) They enforce data uniqueness like a primary key

**35.** What best describes the relationship between a Snowflake Organization and its Accounts?
A) An Organization can group multiple Accounts (e.g., prod/dev, or per-region) for centralized visibility/billing
B) An Account can belong to multiple Organizations simultaneously
C) Organizations and Accounts are the same object with different names
D) Organizations replace the need for Databases

---

## Answer Key — Exam 2

1. B — Data lives in cloud storage managed by the Storage layer, in Snowflake's format.
2. A, C — Business Critical adds Tri-Secret Secure; Enterprise adds multi-cluster + materialized views. (B is false — Standard has the shortest retention; D is false — VPS is the most expensive/isolated tier.)
3. B — User stage = per-user; table stage = tied to one specific table.
4. B — CSV, JSON, Avro, ORC, Parquet, XML are all natively supported.
5. B — FORCE bypasses the duplicate-load file tracking.
6. C — 64 days is the default load metadata retention.
7. B, C — Event-triggered and serverless. (A is false — Snowpipe uses Snowflake-managed compute, not your warehouse; D is false — "near real-time," not a zero-latency guarantee.)
8. B — LATERAL FLATTEN is the correct pattern.
9. B — Task + SYSTEM$STREAM_HAS_DATA() is the standard incremental pipeline pattern.
10. B — Materialized Views require Enterprise edition or higher.
11. B — Scaling out (multi-cluster) addresses concurrency/queuing, not single-query speed.
12. B — Local warehouse cache is lost on suspend; lives on the warehouse itself.
13. B — SUSPEND blocks new queries but lets running ones finish (SUSPEND_IMMEDIATE cancels them).
14. C — Only NOT NULL is enforced.
15. B — CREATE OR REPLACE atomically drops and recreates (data is not preserved in the new table).
16. B, C — Transient and Temporary are capped at 1 day max Time Travel.
17. A — Reader Accounts let non-Snowflake consumers access shared data, provider-managed.
18. B — Cross-region/cloud sharing requires replication (Business Critical+).
19. B — Read-only (SELECT) access by default.
20. B — Difference is discoverability/audience, not underlying technology.
21. B — USERADMIN manages users/roles as a subset of SECURITYADMIN.
22. B — Privileges go to roles, which are granted to users (or other roles).
23. C — PUBLIC is the default role every user has.
24. B — Storage cost grows only for diverged/changed micro-partitions.
25. B — `AT (TIMESTAMP => ...)` is the correct Time Travel query syntax.
26. B — UNDROP TABLE restores a dropped table within retention.
27. B — Fail-safe requires a Snowflake Support request; no role can self-serve it.
28. B — Key-pair authentication is standard for service/programmatic accounts.
29. B — Organization → Account → Database → Schema → Table.
30. A, D — Fixed 7-day period, Support-only recovery. (B is false — only Permanent tables get Fail-safe; C is the true statement, contradicting B, so B is the false option here — select A and D.)
31. C — Result cache serves without needing an active warehouse.
32. B — Query Profile visualizes execution plan and performance metrics.
33. B — DML can be grouped/rolled back in transactions; DDL implicitly commits.
34. B — Clustering keys benefit large tables with well-defined filter columns; not a uniqueness constraint, not primarily a load-speed feature.
35. A — Organizations group multiple Accounts for centralized management/billing.
