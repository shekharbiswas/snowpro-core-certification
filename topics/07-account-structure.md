# Account Structure & Management

## Overview
This domain covers how Snowflake accounts are organized, how access control (RBAC) works, and the mechanics of Cloning, Time Travel, and Fail-safe. This is one of the densest domains on the exam — Time Travel/Fail-safe retention numbers and role hierarchy are very commonly tested.

---

## 1. Object Hierarchy

```
Organization
  └── Account
        └── Database
              └── Schema
                    └── Table / View / Stage / File Format / Stream / Task / etc.
```

- **Organization** — a top-level entity that can group multiple Snowflake accounts (e.g., separate accounts for prod/dev, or per-region) for centralized billing/management visibility.
- **Account** — tied to a specific cloud platform + region. This is the top-level container for your databases, warehouses, users, roles.
- **Database → Schema → Objects** is the standard containment hierarchy for actual data objects.

---

## 2. Role-Based Access Control (RBAC)

Snowflake uses RBAC — permissions are granted to **roles**, and roles are granted to **users** (not permissions directly to users).

### System-Defined Roles (know these — commonly tested)
| Role | Purpose |
|---|---|
| **ACCOUNTADMIN** | Top-level role; combines SYSADMIN + SECURITYADMIN; full control over the account, billing |
| **SECURITYADMIN** | Manages users, roles, and grants (security/access-related administration) |
| **USERADMIN** | Manages users and roles (subset of SECURITYADMIN's scope) |
| **SYSADMIN** | Manages databases, warehouses, and other objects; typically where custom roles should be hierarchically attached |
| **PUBLIC** | Default role every user automatically has; minimal/no privileges by default |

### Role Hierarchy
- Roles can be granted to other roles, forming a hierarchy — a role inherits all privileges of any role granted to it.
- **Best practice:** create custom roles for actual business functions (e.g., `DATA_ANALYST`, `ETL_ROLE`) and grant them up into `SYSADMIN`, rather than granting privileges directly to individual users.
- A user can have multiple roles but operates using **one active role at a time** per session (switchable with `USE ROLE`).

---

## 3. Users & Authentication

Supported authentication methods:
- Username/password
- **MFA** (multi-factor authentication)
- **SSO** (Single Sign-On via SAML2, e.g., Okta, Azure AD)
- **Key-pair authentication** — for programmatic/service accounts (common for CI/CD, drivers, connectors), uses public/private key pairs instead of a password.

---

## 4. Cloning (Zero-Copy Clone)

```sql
CREATE TABLE my_table_clone CLONE my_table;
CREATE DATABASE my_db_clone CLONE my_db;
```

- A **zero-copy clone** creates a new object that initially shares the same underlying micro-partitions as the source — **no additional storage is consumed at the moment of cloning.**
- Storage cost only grows as the clone and the original **diverge** (each new/changed micro-partition after the clone point takes up new space).
- Works on databases, schemas, tables (and a few other object types).
- Extremely useful for spinning up dev/test environments from production data instantly, without doubling storage cost.
- **Clones do NOT automatically stay in sync** with the source after creation — it's a snapshot at that point in time, not a live link.

---

## 5. Time Travel

Time Travel lets you query, clone, or restore data **as it existed at a previous point in time** — even after it's been updated, deleted, or the table itself dropped.

```sql
-- Query data as of a past timestamp
SELECT * FROM my_table AT (TIMESTAMP => '2026-08-01 10:00:00'::TIMESTAMP);

-- Query data as it was before a specific query ran
SELECT * FROM my_table BEFORE (STATEMENT => '<query_id>');

-- Restore a dropped table
UNDROP TABLE my_table;
```

### Retention Periods (memorize these)
| Edition | Default Retention | Max Retention |
|---|---|---|
| Standard | 1 day | 1 day |
| Enterprise+ | 1 day (configurable) | Up to **90 days** |

- Retention is configured per-object via `DATA_RETENTION_TIME_IN_DAYS`.
- Applies to Permanent tables fully; Transient/Temporary tables are capped at **1 day max** regardless of edition.

---

## 6. Fail-safe

- A **separate, non-configurable 7-day period** that begins automatically **after** the Time Travel retention period ends.
- During Fail-safe, data is **only recoverable by contacting Snowflake Support** — it is NOT something you or any role (even ACCOUNTADMIN) can query or restore yourselves via SQL.
- Intended as a last-resort disaster recovery mechanism, not a routine tool.
- **Only Permanent tables have Fail-safe.** Transient and Temporary tables skip it entirely (this is repeated from Domain 5 because it's tested from both the "table types" angle AND the "account structure/data protection" angle).

### Time Travel vs Fail-safe — the #1 tested distinction in this domain
| | Time Travel | Fail-safe |
|---|---|---|
| Who can access | You, via SQL (`AT`/`BEFORE`/`UNDROP`) | Only Snowflake Support |
| Duration | 0–90 days (configurable) | Fixed 7 days, non-configurable |
| When it applies | Immediately after change/drop | After Time Travel period ends |
| Applies to | Permanent (full), Transient/Temp (limited to Time Travel only) | Permanent tables only |

---

## Sample Questions & Answers

**Q1. Which role combines the privileges of both SYSADMIN and SECURITYADMIN?**
A) USERADMIN
B) PUBLIC
C) ACCOUNTADMIN
D) ORGADMIN

**Answer: C — ACCOUNTADMIN.** It sits at the top of the default role hierarchy and has full account-level control, including billing.

---

**Q2. Immediately after creating a zero-copy clone of a large table, how much additional storage is consumed?**
A) The same amount as the original table
B) Approximately none — the clone initially shares the same micro-partitions as the source
C) 50% of the original table's size
D) It depends on the warehouse size used

**Answer: B.** Storage only grows as the clone and original diverge over time (new/changed data creates new micro-partitions specific to each).

---

**Q3. A Permanent table's Time Travel retention period has just ended. Can a user with ACCOUNTADMIN role still recover the data themselves via SQL?**
A) Yes, ACCOUNTADMIN can always recover data at any time
B) No — once Time Travel ends, the data enters Fail-safe, which is only recoverable by contacting Snowflake Support
C) Yes, but only using the UNDROP command
D) No, the data is permanently and immediately unrecoverable

**Answer: B.** Fail-safe is not accessible via SQL/any role — not even ACCOUNTADMIN. It requires a support request to Snowflake.

---

**Q4. Which table type(s) have Fail-safe protection?**
A) All table types equally
B) Only Permanent tables
C) Only Transient tables
D) Only Temporary tables

**Answer: B — Permanent tables only.** Transient and Temporary tables skip Fail-safe entirely (they're designed to trade data protection for lower storage cost).

---

**Q5. What is the maximum Time Travel retention period available, and on which edition?**
A) 7 days, Standard edition
B) 30 days, Enterprise edition
C) 90 days, Enterprise edition or higher
D) Unlimited, Business Critical edition

**Answer: C — up to 90 days, Enterprise edition and above.** Standard edition is capped at 1 day.

## Notes
_(Add your own notes here as you study)_

## Practice Questions Log
_(Track questions you got wrong and why)_
