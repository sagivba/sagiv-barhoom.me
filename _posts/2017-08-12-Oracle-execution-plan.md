---
layout: post
title:  "Oracle execution plan"
author: "Sagiv Barhoom"
date:   2017-08-12
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---

# Oracle Execution Plan

## A. View an execution plan 
While an explain plan is what the optimizer *thinks* will happen when you run your query, the execution plan what *actually happened* when you ran the query.
Remark: *Based on stored object statistics and estimated costs*
1. Generate the plan
    ```sql
    EXPLAIN PLAN FOR
    SELECT * FROM stg_tst_interest_sem_um1_vu a;
    ```

2. Display the plan generated
    ```sql
    SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
    ```


## B. Create an execution plan with the actual run-time statistics
1. Run the query and gather statistics
    ```sql
    SELECT /*+ GATHER_PLAN_STATISTICS */  * 
    FROM stg_tst_interest_sem_um1_vu a;
    ```

2. Display the plan generated
   ### Option 1 - For the last statement executed
   Remark: *sometimes it does not work...*
   ```sql
   SELECT * FROM TABLE (DBMS_XPLAN.DISPLAY_CURSORFORMAT=>'ALLSTATS LAST'));
   ```

   ### Option 2 - For the last statement executed for a specific ```sql_id```
    * (Use this if option 1 did no work)
    a. Get the ```sql_id``` of the statement
       ```sql
       SELECT sql_id, sql_text 
       FROM v$sqlarea 
       WHERE sql_text LIKE '%GATHER_PLAN_STATISTICS%' 
       AND sql_text NOT LIKE '%sqlarea%';
       ```
b. Get execution plan with actual run-time statistics
   *(Replace the "&sql_id" with the actual SQL ID of the statement)*
   ```sql
   SELECT * 
   FROM TABLE (DBMS_XPLAN.DISPLAY_CURSOR(sql_id => '&sql_id',FORMAT=>'ALLSTATS LAST'));
   ```
   BTW: you can add other useful columns using:
   ```sql
   SELECT * 
   FROM TABLE (DBMS_XPLAN.DISPLAY_CURSOR(sql_id => '&sql_id',FORMAT=>'ALLSTATS LAST +cost +bytes'));
   ```
   ### Using ```+outline```
   from https://docs.oracle.com/:
   An outline consists primarily of a set of hints that is equivalent to the optimizer's results for the execution plan generation of a particular SQL statement. 
   When Oracle Database creates an outline, plan stability examines the optimization results using the same data used to generate the execution plan. 
   That is, Oracle Database uses the input to the execution plan to generate an outline, and not the execution plan itself.

So how do we get it ?
  ```sql
    SELECT * 
    FROM TABLE (DBMS_XPLAN.DISPLAY_CURSOR(sql_id => '&sql_id',FORMAT=>'ALLSTATS LAST +outline'));
  ```


## C. Create explain plan with actual run-time statistics using MONITOR report
1. Add MONITOR hint
    ```sql
    SELECT /*+ MONITOR */ COUNT(*), AVG(salary)
    FROM hr.employees
    WHERE department_id = 50;
    ```

2. View recent monitored queries
  (Note the SQL_ID and the SQL_EXEC_ID)
    ```sql
    ALTER SESSION SET NLS_DATE_FORMAT='dd/mm/yyyy hh24:mi:ss';
   
    SELECT sql_id, sql_exec_id, SQL_EXEC_START, SUBSTR(sql_text,1,80)
    FROM V$SQL_MONITOR
    WHERE sql_text LIKE '%MONITOR%'
    ORDER BY SQL_EXEC_START DESC;
    ```

3. Generate the report 
  (Copy and paste the output to notepad)
    ```sql
    SELECT DBMS_SQLTUNE.report_sql_monitor(
                sql_id=>'&sql_id', 
                sql_exec_id=>&sql_exec_id, 
                report_level=>'ALL') as text 
    FROM dual;
    ```
## Understanding the "Predicate Information"
### INDEX UNIQUE SCAN 
From: http://www.dba-oracle.com/
In an index unique scan, oracle reads the index nodes down to the leaf node level and them returns the ROWID for the appropriate single row from the calling SQL. 
Indexes with lots of deleted leaf nodes ("bloated" indexes) will suffer from slower full scan access but with index unique scan access, this index would never benefit from rebuilding or coalescing.

### INDEX FAST FULL SCAN
(From "Ask Tom")
An ```index fast full scan``` reads the ENTIRE index, unsorted, as it exists on disk. 
It is basically using the index as a "skinny" version of the table. 
The query in question would only be accessing attributes in the index 
(we are not using the index as a way to get to the table, we are using the index INSTEAD of the table)
We use multiblock IO and read all of the leaf, branch and the root block. 
We ignore the branch and root blocks and just process the (unordered) data on the leaf blocks.

So the rresults of the query can be resolved by reading the index without touching the table data.

### INDEX RANGE SCAN 
A Range Scan is any scan on an index that is not guaranteed to return zero or one row.
During an index range scan, Oracle accesses adjacent index entries and then uses the ROWID values in the index to retrieve the table rows.

#### INDEX FAST FULL SCAN vs INDEX RANGE SCAN 
From: http://www.dbaref.com/home/dba-routine-tasks/whatisfastfullindexscanvsindexrangescan
* In an index-range scan or full-index scan, index blocks are read one at a time in key order. 
  In a fast full scan, blocks are read in the order in which they appear on the disk and the database can read multiple blocks in a single I/O operation, 
  depending on the value of the server parameter db_file_multiblock_read_count. 
  Reading multiple blocks in a single I/O operation can radically reduce the total number of disk I/Os required, resulting in faster execution of queries.
* The fast full-index scan can be performed in parallel, whereas an index-range scan or full-index scan can be processed only serially. 
  That is, the database can allocate multiple processes to perform a fast full-index scan, but it can use only a single process for traditional index scans. 
  Using parallel query may also substantially reduce query-execution time for systems with multiple CPUs or those with data spread across multiple disk devices.
* Although a full table scan can use parallelism and multiblock read techniques, the number of blocks in a table is typically many times as great as the number 
  of blocks in an index. Therefore, a fast full-index scan usually outperforms an equivalent full-table scan.
