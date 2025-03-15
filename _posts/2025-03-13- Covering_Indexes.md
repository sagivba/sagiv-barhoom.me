---
layout: post
title:  "Proactive Detection of Potential Covering Indexes"
author: "Sagiv Barhoom"
date:   2025-03-13
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---

# Proactive Detection of Potential Covering Indexes

## What is a Covering Index?

A **covering index** is an index that contains all the columns needed to satisfy a query. 
This means the database can retrieve the results directly from the index without accessing the table.

## Why should we use a **Covering Indexes**?

Covering indexes improve `SELECT` query performance by:

- Reducing disk I/O.
- Minimizing the number of data pages read from storage. This can significantly reduce execution time.
- Enhancing query efficiency, leading to performance gains of **2x to 10x**, especially for read-heavy workloads.
- Eliminating the need for table lookups.

## Common Performance Issues Solved
Covering indexes are useful when:
- Queries perform **many table lookups** (high I/O).
- The execution plan shows **INDEX RANGE SCAN + TABLE ACCESS BY ROWID**.
- Queries filter and return only indexed columns.

## Identifying Queries That May Benefit

To find slow queries that might benefit from a covering index, look for:

- Queries with **high logical reads**.<br/>
  Use: `SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR(FORMAT => 'ALLSTATS LAST'));`<br/>
  We are looking for:<br/>
  - Buffers > 100,000
  - `TABLE ACCESS FULL` or `TABLE ACCESS BY INDEX ROWID`
  - `INDEX RANGE SCAN` and  Buffers > 50000
- Execution plans with **TABLE ACCESS BY ROWID** after an index scan.

## Recognizing a Covering Index in Execution Plans

### Simple Example: Improving Query Performance with Covering Indexes

```sql
SELECT student_id, course_id, semester 
FROM grades 
WHERE student_id = :B;
```

#### Without an Index (Full Table Scan)

```
---------------------------------------------------------------------------
| Id  | Operation         | Name   | Rows  | Bytes | Cost(%CPU)| Time     |
---------------------------------------------------------------------------
|   0 | SELECT STATEMENT  |        |    10 |   230 | 60102  (1)| 00:00:03 |
|*  1 |  TABLE ACCESS FULL| GRADES |    10 |   230 | 60102  (1)| 00:00:03 |
---------------------------------------------------------------------------
```
A full table scan is costly, as it reads the entire `grades` table.

#### With a Basic Index on `student_id`

```sql
CREATE INDEX ix_grades ON grades (student_id);
/* Execution plan after creating the index:
------------------------------------------------------------------------------------------------
| Id  | Operation                           | Name      | Rows  | Bytes | Cost(%CPU)| Time     |
------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT                    |           |    10 |   230 |   11   (0)| 00:00:01 |
|   1 |  TABLE ACCESS BY INDEX ROWID BATCHED| GRADES    |    10 |   230 |   11   (0)| 00:00:01 |
|*  2 |   INDEX RANGE SCAN                  | IX_GRADES |    10 |       |    3   (0)| 00:00:01 |
------------------------------------------------------------------------------------------------
*/
```

The index helps, but the database still performs a **TABLE ACCESS BY INDEX ROWID**, meaning it looks up additional columns in the table.

#### With a Covering Index on `student_id, course_id, semester`
```sql
CREATE INDEX ix_grades_2 ON grades (student_id, course_id, semester);
/* Execution plan after using a covering index:
-------------------------------------------------------------------------------
| Id  | Operation        | Name        | Rows  | Bytes | Cost(%CPU)| Time     |
-------------------------------------------------------------------------------
|   0 | SELECT STATEMENT |             |    10 |   230 |    3   (0)| 00:00:01 |
|*  1 |  INDEX RANGE SCAN| IX_GRADES_2 |    10 |   230 |    3   (0)| 00:00:01 |
-------------------------------------------------------------------------------
*/
```
Now, the query is fully served from the index, eliminating the need for table access, resulting in **significant performance improvements**.

### A covering index is in use when:

- The execution plan shows **INDEX RANGE SCAN or INDEX FULL SCAN**.
- There is **no TABLE ACCESS BY ROWID** step.

## **Proactive Detection**

### **Step 1:** Query Analysis
Identifies frequently executed queries that rely on **TABLE ACCESS FULL, INDEX FULL SCAN, or INDEX RANGE SCAN** and use **≤ 4 columns**.

### **Step 2:** Index Evaluation
Checks whether an existing index covers all required columns. If not, it suggests creating or modifying an index.

### **Step 3:** SQL Recommendations
Generates `CREATE INDEX` or `ALTER INDEX` statements to improve performance.

```sql
SET SERVEROUTPUT ON
set linesize 200
ALTER SESSION ENABLE PARALLEL QUERY;
ALTER SESSION FORCE PARALLEL QUERY PARALLEL 4;
ALTER SESSION FORCE PARALLEL DML PARALLEL 4;
ALTER SESSION FORCE PARALLEL DDL PARALLEL 4;

DECLARE
    CURSOR cur_sql IS
  SELECT s.sql_id,
         p.object_name                      AS table_name,
         s.executions,
         s.buffer_gets,
         s.disk_reads,
         COUNT (DISTINCT c.column_name)     AS used_columns,
         s.sql_text
    FROM v$sql vs
         JOIN v$sqlstats s ON vs.SQL_ID = s.SQL_ID
         JOIN v$sql_plan p ON s.sql_id = p.sql_id
         JOIN v$sql_plan_statistics_all ps ON ps.sql_id = s.sql_id
         JOIN dba_tab_columns c ON c.table_name = p.object_name
   WHERE     p.operation IN ('TABLE ACCESS', 'INDEX FULL SCAN')
         --AND vs.PARSING_SCHEMA_NAME NOT IN (SELECT username FROM dba_users WHERE oracle_maintained = 'Y')
         AND vs.PARSING_SCHEMA_NAME LIKE '%NET%'
         AND  c.table_name not like '%SYS%' and c.table_name not like '%$%'
         AND vs.EXECUTIONS > 5000
         AND   ( s.buffer_gets > 100000
              OR ( p.operation = 'INDEX RANGE SCAN' AND s.buffer_gets > 50000)
              )
         AND VS.SQL_TEXT LIKE 'SELECT%'
GROUP BY s.sql_id,
         s.sql_text,
         p.object_name,
         s.executions,
         s.buffer_gets,
         s.disk_reads
  HAVING COUNT (DISTINCT c.column_name) <= 4
ORDER BY s.executions DESC, s.buffer_gets DESC
   FETCH FIRST 30 ROWS ONLY;
   


    v_existing_index_name VARCHAR2(128);
    v_existing_columns VARCHAR2(4000);
    v_suggested_index   VARCHAR2(4000);
    v_new_columns       VARCHAR2(4000);
BEGIN
    DBMS_OUTPUT.PUT_LINE('========= COVERING INDEX ANALYSIS REPORT =========');
    
    FOR rec IN cur_sql LOOP
        DBMS_OUTPUT.PUT_LINE('-----------------------------------------------------');
        DBMS_OUTPUT.PUT_LINE('Analyzing SQL ID: ' || rec.sql_id);
        DBMS_OUTPUT.PUT_LINE('Table: ' || rec.table_name);
        DBMS_OUTPUT.PUT_LINE('SQL: ' || rec.sql_text);
        DBMS_OUTPUT.PUT_LINE('Executions: ' || rec.executions || ', Buffer Gets: ' || rec.buffer_gets || ', Disk Reads: ' || rec.disk_reads);
        DBMS_OUTPUT.PUT_LINE('Results:');

        BEGIN
            SELECT i.index_name, 
                   LISTAGG(c.column_name, ', ') WITHIN GROUP (ORDER BY c.column_position) 
            INTO v_existing_index_name, v_existing_columns
            FROM dba_indexes i
            JOIN dba_ind_columns c ON i.index_name = c.index_name AND i.table_name = c.table_name
            WHERE LOWER(i.table_name) = LOWER(rec.table_name) --  not v_table_name
            GROUP BY i.index_name
            ORDER BY COUNT(c.column_name) DESC
            FETCH FIRST 1 ROW ONLY;
        EXCEPTION
            WHEN NO_DATA_FOUND THEN
                v_existing_index_name := 'None';
                v_existing_columns := 'N/A';
        END;

        v_new_columns := NULL;
        BEGIN
            SELECT LISTAGG(column_name, ', ') WITHIN GROUP (ORDER BY column_name)
            INTO v_new_columns
            FROM dba_tab_columns
           WHERE LOWER(table_name) = LOWER(rec.table_name)
            AND column_name NOT IN (
                SELECT column_name 
                FROM dba_ind_columns
                WHERE LOWER(table_name) = LOWER(rec.table_name)
            );
        EXCEPTION
            WHEN NO_DATA_FOUND THEN
                v_new_columns := NULL;
        END;

        IF v_existing_index_name = 'None' THEN
            v_suggested_index := '  CREATE INDEX idx_covering_' || rec.table_name || ' ON ' || rec.table_name || ' (' || v_new_columns || ');';
            DBMS_OUTPUT.PUT_LINE('  No existing index found.');
            DBMS_OUTPUT.PUT_LINE('  Suggested action: CREATE a new Covering Index.');
            DBMS_OUTPUT.PUT_LINE('  SQL: ' || v_suggested_index);
        
        ELSIF v_new_columns IS NOT NULL THEN
            v_suggested_index := '  ALTER INDEX ' || v_existing_index_name || ' ADD (' || v_new_columns || ');';
            DBMS_OUTPUT.PUT_LINE('  Existing Index Found: ' || v_existing_index_name);
            DBMS_OUTPUT.PUT_LINE('  Indexed Columns: ' || v_existing_columns);
            DBMS_OUTPUT.PUT_LINE('  Suggested action: ALTER existing index to add missing columns.');
            DBMS_OUTPUT.PUT_LINE('  SQL: ' || v_suggested_index);
        
        ELSE
            DBMS_OUTPUT.PUT_LINE('  Existing index ' || v_existing_index_name || ' already covers the query.');
            DBMS_OUTPUT.PUT_LINE('  No action needed.');
        END IF;
        
        DBMS_OUTPUT.PUT_LINE('-----------------------------------------------------');
        DBMS_OUTPUT.NEW_LINE;
    END LOOP;

    DBMS_OUTPUT.PUT_LINE('========= ANALYSIS COMPLETE =========');
END;
/

```

