# Performance Optimization

## Overview
This domain covers virtual warehouse sizing/scaling, caching, clustering, and diagnosing slow queries. Expect several questions testing whether you know WHEN to scale up vs scale out, and how Snowflake's caching layers differ.

---

## 1. Virtual Warehouses — Sizing

Warehouses come in T-shirt sizes: **X-Small, Small, Medium, Large, X-Large, 2X-Large ... up to 6X-Large.**

- Each size up **doubles** the compute (and credit cost per hour) of the previous size.
- **Scaling UP (bigger size)** = more compute power per query → speeds up complex/large individual queries.
- **Scaling OUT (multi-cluster)** = adding more clusters of the *same* size → handles more concurrent queries/users, does NOT make an individual query run faster.

**Exam trap:** "Query is slow" → consider scaling **up**. "Many users are queuing/waiting" (concurrency problem) → consider scaling **out** (multi-cluster warehouse), not up.

---

## 2. Multi-Cluster Warehouses (Concurrency)

Available on Enterprise edition and above.

```sql
CREATE WAREHOUSE my_wh
  WAREHOUSE_SIZE = 'MEDIUM'
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 4
  SCALING_POLICY = 'STANDARD';
```

- `MIN_CLUSTER_COUNT` / `MAX_CLUSTER_COUNT`: Snowflake automatically starts/stops additional clusters within this range based on query load.
- **Scaling Policy:**
  - `STANDARD` (default) — favors starting new clusters quickly to minimize queuing.
  - `ECONOMY` — favors conserving credits, may let some queuing happen before starting a new cluster.

---

## 3. Auto-Suspend / Auto-Resume

```sql
CREATE WAREHOUSE my_wh
  WAREHOUSE_SIZE = 'SMALL'
  AUTO_SUSPEND = 60         -- seconds of inactivity before suspending
  AUTO_RESUME = TRUE;
```

- **Auto-suspend** stops billing once idle for the configured time — a key cost-control lever.
- **Auto-resume** automatically starts the warehouse again the moment a new query is submitted (usually near-instant for warehouse startup).
- Short auto-suspend = saves credits but more frequent cold-starts; long auto-suspend = fewer cold-starts but wastes credits during idle time. There's a cost/performance trade-off — the exam likes to test that you understand this is a trade-off, not a "set it and forget it."

---

## 4. Caching Layers (commonly confused — know all three!)

| Cache | What it stores | Lives on | Persists after warehouse suspend? |
|---|---|---|---|
| **Result Cache** | Full result set of a previously run query (identical query, same data unchanged) | Cloud Services layer | Yes — up to 24 hours, doesn't need a running warehouse at all |
| **Warehouse (Local Disk/SSD) Cache** | Raw data pages read from storage, cached on the warehouse's local SSD | The virtual warehouse itself | No — lost when warehouse suspends |
| **Metadata Cache** | Table statistics (min/max values, row counts) used for pruning | Cloud Services layer | Yes |

**Exam trap:** If you re-run the *exact same query* with no data changes, Snowflake can return the answer instantly from the **Result Cache** — without even starting/using a virtual warehouse. This is a very commonly tested point.

---

## 5. Clustering Keys & Re-clustering

- By default, Snowflake automatically organizes micro-partitions based on data load order (natural clustering).
- For very large tables where query filters don't align well with load order, you can define an explicit **clustering key**:
```sql
ALTER TABLE my_table CLUSTER BY (date_column);
```
- Snowflake performs **automatic re-clustering** in the background to keep data organized per the clustering key as new data arrives (uses serverless compute, billed separately).
- Clustering mainly helps **pruning** — Snowflake can skip more micro-partitions when scanning, since matching values are co-located.
- **Only worth it for large tables (multi-TB)** with a clear, frequently-filtered column. On smaller tables it adds cost with little benefit.

---

## 6. Query Profile

- The **Query Profile** (in Snowsight or classic UI) is the primary tool for diagnosing a slow query — shows a visual execution plan with time spent per operator (scan, join, aggregate, etc.), bytes scanned, spillage to disk, and partition pruning stats.
- Look for:
  - **Partition pruning ratio** — low pruning = query is scanning far more data than needed (missing/ineffective clustering, or filter not on a well-clustered column).
  - **Spilling to local/remote storage** — means the warehouse ran out of memory for that operation (often fixable by using a larger warehouse for that query).

---

## 7. Resource Monitors

- Used to **track and control credit usage**, not query performance directly, but frequently grouped in this domain.
```sql
CREATE RESOURCE MONITOR my_monitor
  WITH CREDIT_QUOTA = 1000
  TRIGGERS ON 80 PERCENT DO NOTIFY
           ON 100 PERCENT DO SUSPEND;
```
- Can trigger actions like `NOTIFY`, `SUSPEND` (lets running queries finish, blocks new ones), or `SUSPEND_IMMEDIATE` (cancels running queries too).

---

## Sample Questions & Answers

**Q1. A single complex query is taking too long to run, but there's no concurrency issue (only one user running queries). What's the best first fix?**
A) Increase MAX_CLUSTER_COUNT
B) Increase the warehouse size (scale up)
C) Enable auto-suspend
D) Create a materialized view immediately

**Answer: B.** Scaling up gives more compute per query, which helps a single large/complex query run faster. Multi-cluster (scaling out) only helps when multiple queries are competing/queuing — not relevant here.

---

**Q2. You re-run the exact same SELECT query twice within a few minutes, and the underlying table hasn't changed. What happens the second time?**
A) The query re-scans the table but uses the warehouse's local SSD cache
B) The result is returned instantly from the Result Cache, without needing an active warehouse
C) Snowflake always re-executes every query fully for consistency
D) It depends on the warehouse size

**Answer: B.** This is the Result Cache — served entirely from the Cloud Services layer for up to 24 hours, no virtual warehouse needs to even be running.

---

**Q3. What is the main purpose of a clustering key?**
A) To enforce uniqueness like a primary key
B) To improve partition pruning by keeping related data co-located in micro-partitions
C) To speed up data loading
D) To reduce storage costs

**Answer: B.** Clustering keys are about query performance via better pruning on large tables — not uniqueness constraints (Snowflake doesn't enforce those anyway) and not primarily about load speed or storage cost.

---

**Q4. Which scaling policy favors minimizing query queuing over conserving credits in a multi-cluster warehouse?**
A) ECONOMY
B) STANDARD
C) AGGRESSIVE
D) BALANCED

**Answer: B — STANDARD.** It's the default and starts additional clusters quickly to avoid queuing. ECONOMY is the more cost-conscious option that tolerates some queuing before scaling out. (AGGRESSIVE and BALANCED aren't real Snowflake scaling policy names.)

---

**Q5. In Query Profile, you notice heavy "spillage to remote storage" on one step. What does this indicate?**
A) The warehouse ran out of memory for that operation and had to spill intermediate results to disk/remote storage
B) The query is using the result cache
C) The table needs a clustering key
D) The warehouse is auto-suspended

**Answer: A.** Spilling means the operation (e.g., a large sort or join) didn't fit in the warehouse's available memory. Using a larger warehouse (more memory per node) is a common fix.

## Notes
_(Add your own notes here as you study)_

## Practice Questions Log
_(Track questions you got wrong and why)_
