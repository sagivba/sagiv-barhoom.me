---
layout: post
title:  "GATHER STATS HINT 101"
author: "Sagiv Barhoom"
date:   2017-08-12
categories: ORACLE 
background: '/img/posts/randomnace.jpg'
---

# some note on `GATHER_STATS`
 
The `GATHER_PLAN_STATISTICS` hint is used to instruct Oracle to gather statistics for the specified table. 
By providing this hint, we are explicitly telling Oracle to collect fresh statistics for the table. 
This can help the query optimizer make better decisions when generating query execution plans.

```sql
SELECT /*+ GATHER_PLAN_STATISTICS */ * FROM my_table_name;
```
We can also use: 
```sql
EXEC DBMS_STATS.GATHER_TABLE_STATS('SCHEMA_NAME', 'my_table');
```

## last analyzed table
To check when statistics for a table were last gathered:
```sql
SELECT owner, table_name, last_analyzed
FROM dba_tables -- or user_tables or all_tables
WHERE table_name = 'MY_TABLE_NAME';
```

## Data about the histogram of columns
A histogram is a type of column statistic that provides detailed information about the distribution of data in a column.
```sql
SELECT COLUMN_NAME,
       NOTES,
       tcs.NUM_BUCKETS,
       tcs.NUM_DISTINCT,
       HISTOGRAM
  FROM DBA_TAB_COL_STATISTICS tcs -- or USER_TAB_COL_STATISTICS or ALL_TAB_COL_STATISTICS
 WHERE TABLE_NAME = 'MY_TABLE_NAME';
```

## `DBMS_STATS` examples
`DBMS_STATS` is a package that allows us to manually collect, modify, and manage statistics for database objects.
Here are some examples of usage:

### Collect Statistics for a Table|Index
```sql
EXEC DBMS_STATS.GATHER_TABLE_STATS('MY_SCHEMA', 'MY_TABLE');
EXEC DBMS_STATS.GATHER_INDEX_STATS('MY_SCHEMA', 'MY_INDEX');
```
### Estimate Statistics Without Gathering
This will estimate statistics based on a sample of the data - useful for huge tables.
```sql
EXEC DBMS_STATS.GATHER_TABLE_STATS('YOUR_SCHEMA', 'YOUR_TABLE', estimate_percent => 10);;
```

### Lock and Unlock Statistics
we can lock statistics for a table to prevent them from being automatically overwritten.
```sql
EXEC DBMS_STATS.LOCK_TABLE_STATS('MY_SCHEMA', 'MY_TABLE');
EXEC DBMS_STATS.UNLOCK_TABLE_STATS('MY_SCHEMA', 'MY_TABLE');
```
### Delete Statistics
```sql
EXEC DBMS_STATS.DELETE_TABLE_STATS('MY_SCHEMA', 'MY_TABLE');
EXEC DBMS_STATS.DELETE_INDEX_STATS('MY_SCHEMA', 'MY_TABLE');
;
```
### gather histogram stats
```sql
BEGIN
  DBMS_STATS.GATHER_TABLE_STATS ( 
    ownname    => 'SYSTEM',
    tabname    => 'USERS_DEMO',
    method_opt => 'FOR COLUMNS STATUS'
);
END;
```

## The `DBMS_XPLAN.DISPLAY_CURSOR` package
We can  see a detailed output of the execution statistics.
This can help us analyze the query's performance and identify improvement areas.
The information  on CPU and I/O statistics, memory usage, and time-related metrics.
```SQL
SELECT *
FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR(sql_id=>'dhghrrgrwba46',format => 'ALLSTATS LAST'));
```
The format => 'ALLSTATS LAST' parameter ensures that all available statistics for the last execution are displayed.

```
SQL_ID  dhghrrgrwba46, child number 0
-------------------------------------
SELECT  * FROM my_table_name

Plan hash value: 3956160934

----------------------------------------------------------------------------------------------------
| Id  | Operation         | Name         | Starts | E-Rows | A-Rows |   A-Time   | Buffers | Reads |
------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT  |              |      1 |        |   1076 |00:00:00.01 |      33 |     9 |
|   1 |  TABLE ACCESS FULL| my_table_name|      1 |   220  |   3300 |00:00:00.01 |      33 |     9 |
----------------------------------------------------------------------------------------------------

Note
-----
   - dynamic sampling used for this statement (level=2)
   - statistics feedback used for this statement
```
In the above example the actual rows (`A-Row`) are greater by the scale of 10 then the estemated rows (`E-Rows`).
So gathering statistics  on that table may improve the performance of the above query.

