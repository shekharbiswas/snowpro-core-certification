# SnowPro Core — Practice Exam 1

**Total Questions:** 35
**Question Types:** Multiple Choice (select 1), Multiple Select (select 2+, as noted)
**Time Limit:** 65 minutes
**Passing Guidance:** Aim for 75%+ (real exam pass mark is ~750/1000 scaled score) before sitting the actual exam

---

**Instructions:** Answer all questions before checking the answer key at the bottom. Questions marked **(Multiple Select)** have more than one correct answer — select all that apply. All others are single-answer Multiple Choice.

---

**1.** Which layer of Snowflake's architecture is responsible for query parsing, optimization, and access control?
A) Database Storage Layer
B) Query Processing Layer
C) Cloud Services Layer
D) Virtual Warehouse Layer

**2.** What is a micro-partition in Snowflake?
A) A manually created index
B) A small, immutable unit of columnar storage (50–500 MB uncompressed) that Snowflake automatically creates
C) A backup copy of a table
D) A type of virtual warehouse

**3. (Multiple Select)** Which of the following are true about Snowflake virtual warehouses? (Select 2)
A) Warehouses share compute resources with each other by default
B) Each warehouse is an independent compute cluster
C) Billing is calculated per-second with a 60-second minimum
D) Warehouse size affects storage cost

**4.** Which Snowflake edition is the minimum required for multi-cluster warehouses?
A) Standard
B) Enterprise
C) Business Critical
D) Virtual Private Snowflake (VPS)

**5.** What is the purpose of a named external stage?
A) To store query results temporarily
B) To point to a customer-owned cloud storage location (e.g., S3 bucket) that Snowflake can read from
C) To define a virtual warehouse
D) To enforce a primary key constraint

**6.** Which command uploads a local file into an internal stage?
A) COPY INTO
B) PUT
C) GET
D) LOAD

**7.** What does `ON_ERROR = 'CONTINUE'` do in a COPY INTO statement?
A) Aborts the entire load on the first error
B) Skips rows with errors and continues loading the remaining valid rows
C) Retries the failed row up to 3 times
D) Pauses the load and waits for manual intervention

**8.** What triggers a Snowpipe load?
A) A scheduled Task running every hour
B) A manual COPY INTO command
C) Cloud provider event notifications when new files arrive in a stage (or a REST API call)
D) A user querying the table

**9. (Multiple Select)** Which of the following are true about Snowpipe? (Select 2)
A) It uses your own virtual warehouse for compute
B) It uses Snowflake-managed serverless compute
C) It is designed for near real-time, continuous loading
D) It requires manual execution for every file

**10.** In `SELECT raw_data:name::STRING FROM my_table`, what does the `::` operator do?
A) Accesses a nested key
B) Casts the VARIANT value to an explicit data type
C) Flattens an array
D) Concatenates two strings

**11.** Which table function is used to unnest a VARIANT array into multiple rows?
A) UNNEST()
B) FLATTEN()
C) EXPAND()
D) SPLIT()

**12.** A Stream records an UPDATE to a row. How is this represented?
A) A single row with METADATA$ACTION = 'UPDATE'
B) A DELETE row followed by an INSERT row, both with METADATA$ISUPDATE = TRUE
C) Only the new value is shown, old value discarded
D) Streams cannot capture UPDATE operations

**13.** What is the main difference between a standard View and a Materialized View?
A) Views are faster because they cache automatically
B) Materialized Views physically store precomputed results and are refreshed automatically; standard Views run the query live each time
C) Materialized Views are available on all editions
D) Views cannot be queried directly

**14.** A single, complex query is running slowly, with no other users querying concurrently. What is the most appropriate fix?
A) Increase MAX_CLUSTER_COUNT
B) Scale up to a larger warehouse size
C) Lower AUTO_SUSPEND
D) Create a Reader Account

**15.** Which caching layer allows Snowflake to return a query result almost instantly, without needing an active virtual warehouse, if the query and underlying data are unchanged?
A) Warehouse (local disk/SSD) cache
B) Metadata cache
C) Result cache
D) Cloud Services cache

**16. (Multiple Select)** Which of the following are valid Snowflake multi-cluster warehouse scaling policies? (Select 2)
A) STANDARD
B) ECONOMY
C) AGGRESSIVE
D) BALANCED

**17.** What is the primary purpose of a clustering key?
A) To enforce row uniqueness
B) To improve partition pruning by keeping related data co-located
C) To speed up data loading specifically
D) To reduce compute costs during loading

**18.** In Query Profile, "spillage to remote storage" on an operator typically indicates:
A) The result cache was used
B) The warehouse ran out of memory for that operation
C) The table needs a primary key
D) A clustering key is missing

**19.** Which constraint type does Snowflake actually enforce (reject invalid data)?
A) PRIMARY KEY
B) FOREIGN KEY
C) UNIQUE
D) NOT NULL

**20.** What happens to previously-removed data after running `TRUNCATE TABLE my_table;`?
A) It is permanently and immediately unrecoverable
B) The table structure remains and the removed data may still be recoverable via Time Travel within the retention window
C) TRUNCATE is not supported in Snowflake
D) The table is automatically dropped

**21. (Multiple Select)** Which table types do NOT have Fail-safe protection? (Select 2)
A) Permanent
B) Transient
C) Temporary
D) External

**22.** Which SQL statement performs an "upsert" (insert new rows, update matching existing rows) in a single statement?
A) UPSERT INTO
B) MERGE
C) REPLACE INTO
D) COPY INTO

**23.** What is true about DDL statements (e.g., CREATE, ALTER) inside a transaction block?
A) They can be rolled back like DML
B) They implicitly commit and cannot be rolled back
C) They are not allowed inside a BEGIN/COMMIT block
D) They require ACCOUNTADMIN role

**24.** What is the defining characteristic of Snowflake's Secure Data Sharing?
A) Data is copied to the consumer's account
B) No data is copied — consumers query the same underlying data live via metadata pointers
C) It requires exporting to CSV first
D) It only works within the same Snowflake account

**25.** Who pays for the storage cost of data accessed via a Share?
A) The consumer account
B) The provider account (consumer only pays their own compute to query it)
C) Both split the cost equally
D) Snowflake absorbs the storage cost

**26.** How can a provider share data with a partner who does not have their own Snowflake account?
A) It's not possible
B) By creating a Reader Account for the partner
C) By exporting to Parquet and emailing it
D) Through the Marketplace only

**27.** Standard Secure Data Sharing (without replication) works:
A) Across any region and any cloud provider
B) Only within the same region and same cloud platform
C) Only within Business Critical edition
D) Only for Permanent tables

**28.** Which system role combines the privileges of both SYSADMIN and SECURITYADMIN?
A) USERADMIN
B) PUBLIC
C) ACCOUNTADMIN
D) ORGADMIN

**29.** Immediately after creating a zero-copy clone, how much extra storage is used?
A) The full size of the source table
B) Approximately none — the clone shares the source's micro-partitions until data diverges
C) 50% of the source size
D) Depends on the clone's warehouse size

**30.** After a Permanent table's Time Travel period ends, who can recover the data during Fail-safe?
A) Any user with SYSADMIN
B) Only Snowflake Support, via a support request
C) The table owner, via UNDROP
D) No one — data is unrecoverable

**31.** What is the maximum Time Travel retention period, and on which edition is it available?
A) 7 days, Standard
B) 30 days, Enterprise
C) 90 days, Enterprise or higher
D) Unlimited, Business Critical

**32. (Multiple Select)** Which of the following are valid Snowflake authentication methods? (Select 3)
A) Username/Password
B) SSO (SAML2)
C) Key-pair authentication
D) IP-only authentication (no credentials)

**33.** In the object hierarchy, which is the correct top-down order?
A) Database → Organization → Account → Schema
B) Organization → Account → Database → Schema
C) Account → Organization → Schema → Database
D) Schema → Database → Account → Organization

**34.** A COPY INTO transformation SELECT clause supports which of the following?
A) Full joins across multiple staged files
B) Simple row-level expressions like column selection, casts, and functions
C) Window functions and aggregations
D) Subqueries against other tables

**35.** What does `AUTO_SUSPEND = 60` on a warehouse configure?
A) The warehouse resizes automatically after 60 queries
B) The warehouse suspends (stops billing) after 60 seconds of inactivity
C) The warehouse runs for a maximum of 60 minutes
D) Queries time out after 60 seconds

---

## Answer Key — Exam 1

1. C — Cloud Services Layer handles parsing, optimization, and access control.
2. B — Micro-partitions are automatic, immutable columnar storage units.
3. B, C — Warehouses are independent clusters; billed per-second (60s min). (A is false — they don't share compute; D is false — size affects compute cost, not storage.)
4. B — Multi-cluster warehouses require Enterprise edition or higher.
5. B — External stages point to customer-owned cloud storage.
6. B — PUT uploads local files to an internal stage.
7. B — CONTINUE skips bad rows, loads the rest.
8. C — Snowpipe is event-driven (or REST API triggered).
9. B, C — Serverless compute, near real-time. (A is false; D is false — that's manual COPY INTO, not Snowpipe.)
10. B — `::` casts VARIANT to an explicit type.
11. B — FLATTEN() unnests arrays/objects into rows.
12. B — Streams show UPDATE as paired DELETE+INSERT with METADATA$ISUPDATE = TRUE.
13. B — Materialized Views store precomputed results; Views run live.
14. B — Scale up for single complex queries; scale out is for concurrency.
15. C — Result Cache serves instantly without needing a running warehouse.
16. A, B — STANDARD and ECONOMY are the two valid policies.
17. B — Clustering keys improve pruning via data co-location.
18. B — Spillage means insufficient memory for that operation.
19. D — NOT NULL is the only enforced constraint.
20. B — TRUNCATE keeps structure; data may be Time Travel recoverable.
21. B, C — Transient and Temporary tables skip Fail-safe.
22. B — MERGE performs upsert logic.
23. B — DDL implicitly commits; cannot be rolled back.
24. B — No data copying; live metadata-based access.
25. B — Provider pays storage; consumer pays only their own compute.
26. B — Reader Accounts let non-Snowflake partners access shared data.
27. B — Same region/cloud only, without replication.
28. C — ACCOUNTADMIN combines SYSADMIN + SECURITYADMIN.
29. B — Zero-copy clones share micro-partitions initially; storage grows only on divergence.
30. B — Fail-safe recovery requires contacting Snowflake Support.
31. C — Up to 90 days, Enterprise edition or higher.
32. A, B, C — Password, SSO, key-pair are valid; IP-only auth isn't a Snowflake auth method.
33. B — Organization → Account → Database → Schema.
34. B — Only simple row-level expressions are supported in COPY INTO transforms.
35. B — AUTO_SUSPEND is measured in seconds of inactivity before suspending.
