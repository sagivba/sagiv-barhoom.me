---
layout: post
title:  "Row-by-Row Pitfalls in Oracle"
author: "Sagiv Barhoom"
date:   2025-03-02
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---


# Row-by-Row Pitfalls in Oracle

Identifying and Resolving Inefficient Row-by-Row Table Access in Oracle Queries

In Oracle databases, a common performance issue occurs when queries rely on Index Range Scans combined with Table Access by ROWID. 
This happens when:
- The query scans an index to find matching ROWIDs.
- For each ROWID, Oracle accesses the table to fetch additional columns.
- The process repeats for every matching row, leading to repeated table lookups.

This row-by-row table access can significantly degrade performance, especially if the query needs to retrieve a large number of rows.

## How to Detect This Pattern in a Specific Query
When examining an Execution Plan, We will see the following pattern:

- `INDEX RANGE SCAN` — Oracle scans the index.
- `TABLE ACCESS BY ROWID` — Oracle fetches each row from the table using the ROWID found in the index.

| Id  | Operation               | Name         |
|---- |-------------------------|--------------|
|  0  | SELECT STATEMENT        |              |
|  1  |  TABLE ACCESS BY ROWID  | MY_TABLE     |
|  2  |   INDEX RANGE SCAN      | MY_INDEX     |

This access path is fine for a small number of rows, but it becomes very inefficient when handling large result sets.


## Query to Find the 30 Most Frequent Queries with This Pattern
We can run the following SQL to identify the 30 most frequently executed queries in your database that suffer from this Index Range Scan + Table Access by ROWID pattern:
```sql
WITH plan_with_steps AS (
    SELECT s.sql_text,    s.executions,    s.fetches,
           s.disk_reads,  s.buffer_gets,   s.rows_processed,
           s.cpu_time,    s.elapsed_time,  s.sql_id,
           ROW_NUMBER() OVER (PARTITION BY s.sql_id ORDER BY p.id) AS step_number,
           p.operation,   p.options
    FROM v$sql s JOIN v$sql_plan p ON s.sql_id = p.sql_id
    WHERE p.operation = 'TABLE ACCESS'
      AND p.options = 'BY INDEX ROWID'
      AND EXISTS (
          SELECT 1
          FROM v$sql_plan p2
          WHERE p2.sql_id = p.sql_id AND p2.id < p.id
            AND p2.operation = 'INDEX'
            AND p2.options LIKE '%RANGE SCAN%'
      )
)
SELECT sql_text,
       executions,      fetches,    disk_reads,   buffer_gets,
       rows_processed,  cpu_time,   elapsed_time, sql_id
FROM plan_with_steps
WHERE step_number = 1
ORDER BY executions DESC
FETCH FIRST 30 ROWS ONLY;
```
This query:
- Finds all queries with TABLE ACCESS BY ROWID.
- Ensures they follow an INDEX RANGE SCAN.
- Sorts by execution count, showing the 30 most frequently executed queries.

## Solutions 

- We can add covering indexes to eliminate the need for table lookups.
  A **covering index** is a database index that contains *all* the columns required to fulfill a specific query.
  Using  covering index will retrieve all the necessary data directly from the index without accessing the table.
- Tuning the queries to reduce the number of columns retrieved.
- Evaluating whether a Full Table Scan would be more efficient.
  I only use /+full(test)/ for one-time processes, like data conversion. Overall, I'm not fan of execution plan hints.
