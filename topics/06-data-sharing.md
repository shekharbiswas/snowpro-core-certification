# Secure Data Sharing

## Overview
This domain covers how Snowflake accounts share live data with each other **without copying or moving it**. It's a signature Snowflake feature and shows up reliably on the exam — know the terminology (provider/consumer, share, reader account) cold.

---

## 1. The Core Idea: No Data Copying

Traditional data sharing = export a file, send it, other party imports it (stale the moment it's sent, duplicated storage).

**Snowflake Secure Data Sharing** = the consumer queries the *same underlying data* live, through metadata pointers — no data is copied, moved, or duplicated. The provider's storage is the only copy; consumers just get read access via metadata.

- Because no data moves, the **consumer pays zero storage cost** for shared data — they only pay compute (their own warehouse) to query it.
- The **provider still pays for storage** as normal.
- Changes made by the provider are visible to consumers **immediately** — it's a live view, not a stale extract.

---

## 2. Key Roles: Provider & Consumer

- **Provider** — the account that owns the data and creates a Share to expose it.
- **Consumer** — the account that receives access to a Share and queries the data from it.

A single Share can have multiple consumer accounts.

---

## 3. Shares — The Object That Makes This Work

```sql
-- Provider side
CREATE SHARE my_share;
GRANT USAGE ON DATABASE my_db TO SHARE my_share;
GRANT USAGE ON SCHEMA my_db.my_schema TO SHARE my_share;
GRANT SELECT ON TABLE my_db.my_schema.my_table TO SHARE my_share;
ALTER SHARE my_share ADD ACCOUNTS = consumer_account_id;
```

```sql
-- Consumer side
CREATE DATABASE shared_db FROM SHARE provider_account.my_share;
SELECT * FROM shared_db.my_schema.my_table;  -- queries live data, no copy
```

- A Share is essentially a named collection of **grants** (what objects) + **accounts** (who can access) bundled together.
- Only `SELECT` (read) access is shared — consumers cannot modify the provider's data.
- Consumers CAN join shared data with their own local tables in queries.

---

## 4. Reader Accounts

- What if the consumer **doesn't have their own Snowflake account**? The provider can create a **Reader Account** on the consumer's behalf.
- The provider manages and pays for the Reader Account's compute (it's essentially a "lite" Snowflake account created and controlled by the provider specifically to let a non-Snowflake customer view shared data).
- Common for sharing data with smaller partners/customers who don't want to become a full paying Snowflake customer.

---

## 5. Cross-Region / Cross-Cloud Sharing

- Standard Secure Data Sharing only works **within the same region and same cloud platform**.
- To share across regions or across cloud providers (e.g., AWS us-east-1 provider sharing to an Azure West Europe consumer), the provider must set up **Database Replication** first — replicating the database into the target region/cloud, and then sharing from the replica.
- This requires **Business Critical edition or higher** for replication features.

**Exam trap:** Don't assume you can directly share across regions/clouds without replication — that's a very commonly tested distinction.

---

## 6. Snowflake Marketplace / Data Exchange

- **Snowflake Marketplace** — a public marketplace where providers can list datasets for any Snowflake customer to discover and access (some free, some paid), without needing a private, pre-arranged Share agreement.
- **Data Exchange** — a private version of the same concept, for sharing within a defined group (e.g., a company and its approved partners/vendors only), not publicly listed.
- Both build on top of the same underlying Secure Data Sharing technology (Shares) — they're really about *discovery and distribution* rather than a different sharing mechanism.

---

## Sample Questions & Answers

**Q1. When Account A shares a table with Account B using Secure Data Sharing, what happens to storage costs?**
A) Both accounts pay for storage of the shared data
B) Only the consumer (Account B) pays for storage
C) Only the provider (Account A) pays for storage; the consumer pays no storage cost for shared data
D) Storage cost is split automatically based on usage

**Answer: C.** Since no data is copied, the provider remains the sole owner of the physical storage. The consumer only incurs compute costs when querying the shared data via their own warehouse.

---

**Q2. A provider wants to share data with a partner who does NOT have their own Snowflake account. What's the solution?**
A) It's not possible — the partner must sign up for Snowflake first
B) The provider creates a Reader Account for the partner
C) The provider must export the data as a CSV and send it manually
D) The partner can access it via the Snowflake Marketplace automatically

**Answer: B — Reader Account.** The provider creates and manages (and pays for) a Reader Account specifically so a non-Snowflake customer can query the shared data.

---

**Q3. Can a Snowflake account in AWS us-east-1 directly share data with an account on Azure West Europe using standard Secure Data Sharing?**
A) Yes, sharing works across any region/cloud automatically
B) No — this requires setting up Database Replication first (Business Critical edition+), then sharing the replica
C) No, cross-cloud sharing is not supported under any circumstances
D) Yes, but only for Standard edition accounts

**Answer: B.** Standard sharing is same-region/same-cloud only. Cross-region or cross-cloud sharing needs replication set up first, and that capability requires Business Critical edition or above.

---

**Q4. What level of access does Secure Data Sharing grant to a consumer by default?**
A) Full read/write access to the provider's data
B) Read-only (SELECT) access — consumers cannot modify the provider's shared data
C) DDL access to alter the shared table's structure
D) Access is determined per-query by the consumer

**Answer: B.** Shares grant read access only. Consumers can query and join shared data with their own tables, but cannot INSERT/UPDATE/DELETE or ALTER the provider's objects.

---

**Q5. What is the main difference between the Snowflake Marketplace and a private Data Exchange?**
A) Marketplace uses a different underlying technology than Shares
B) Marketplace is publicly discoverable by any Snowflake customer; Data Exchange is a private, invite-only version for a defined group
C) Data Exchange is free while Marketplace always charges
D) Only Data Exchange supports live, non-copied data

**Answer: B.** Both are built on the same Secure Data Sharing mechanism — the difference is about audience and discoverability (public marketplace vs. private, curated group), not the underlying sharing technology.

## Notes
_(Add your own notes here as you study)_

## Practice Questions Log
_(Track questions you got wrong and why)_
