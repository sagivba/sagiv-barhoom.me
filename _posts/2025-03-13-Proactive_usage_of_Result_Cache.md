---
layout: post
title:  "Proactive using Result Cache"
author: "Sagiv Barhoom"
date:   2025-03-13
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---

# Proactive using `Result Cache`

## What is Oracle Result Cache?
The **Oracle Result Cache** is a memory feature that **stores query results** in the database's shared memory (SGA).  
This allows frequently executed queries to **return results instantly**, without re-executing them or accessing disk.

**When should you use Result Cache?**
- Queries that return **static or rarely changing data**.
- Queries that **run frequently** but return the **same results**.
- Queries that are **fast but still consume resources**.

⚠ Warning: Excessive use of Result Cache can lead to memory contention and unexpected cache invalidations.
It is highly recommended to consult with a DBA before implementing Result Cache extensively.
Additionally, Result Cache may require database tuning, such as increasing SGA size and buffer allocation to efficiently accommodate cached results.

## Identifying Queries for Result Cache Optimization
The following SQL identifies queries that could benefit from **Result Cache**.  
It selects queries that:
- **Run frequently (`executions > 1000`)**.
- **Have a low buffer cache impact (`buffer_gets / executions < 1000`)**.
- **Execute quickly (`elapsed_time / executions < 0.5s`)**.
- **Are not from Oracle system users**.

```sql
SELECT sql_id,
       executions,
       buffer_gets,
       disk_reads,
       ROUND(elapsed_time / NULLIF(executions, 0), 2) AS avg_elapsed_time_per_exec,
       ROUND(buffer_gets / NULLIF(executions, 0), 2) AS avg_buffer_gets_per_exec,
       parsing_schema_name,
       CASE
           WHEN executions > 10000
                AND buffer_gets / NULLIF(executions, 0) < 500
                AND elapsed_time / NULLIF(executions, 0) < 500000
           THEN 'Good candidate for Result Cache'
           ELSE 'Requires further analysis'
       END AS recommendation,
       q'[SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR(']' || sql_id || q'[', NULL, 'ALL'));]' AS get_plan,
       sql_text
FROM v$sql
WHERE parsing_schema_name NOT IN (SELECT username FROM dba_users WHERE oracle_maintained = 'Y')
      AND executions > 1000 -- Queries that run frequently
      AND elapsed_time / NULLIF(executions, 0) < 500000 -- Avoid long-running queries
      AND buffer_gets / NULLIF(executions, 0) < 1000 -- Avoid queries that scan too much data
ORDER BY executions DESC
FETCH FIRST 50 ROWS ONLY;
```

---

## How Does This Query Improve Performance?
1. **Reduces CPU & Memory Load** – Result Cache prevents redundant calculations.
2. **Eliminates Repeated Disk I/O** – Cached results are served from memory instead of re-reading from disk.
3. **Speeds Up Response Time** – Queries return instantly instead of executing again.

---

## How to Apply Result Cache?
Once you find a good candidate query, add **Result Cache Hint**:
```sql
SELECT /*+ RESULT_CACHE */ COUNT(*)
FROM orders
WHERE order_status = 'COMPLETED';
```
Or, enable **Result Cache for PL/SQL functions**:
```sql
CREATE OR REPLACE FUNCTION get_discount_rate(p_customer_id NUMBER)
RETURN NUMBER RESULT_CACHE
AS ...
```

## Conclusion
Using **Result Cache** in Oracle can dramatically **improve query performance**, especially for frequently executed **read-heavy queries**.  
By running our SQL analysis, we can **proactively identify** queries that would benefit from caching and reduce overall system load.


