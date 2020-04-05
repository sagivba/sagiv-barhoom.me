---   
layout: post
title: "Gather information for performance tunning"
subtitle: "Simple usage an examples - of user and profiles management"
date: 2020-02-17
background: '/img/posts/06.jpg'
categories: Oracle
---  
# Gather info for performance tunning
## if you know what is running slow:
1. Collect select statment from the programmer
2. Get explain plans (text file)
3. Get plan statistics (html file)
4. Get ASH reprot (html file)
5. Get AWR reprot (html file)


## if you *do not* know what is running slow:
1. Run the program with the programmer
2. Get ASH reprot (html file)
3. Collect select statment from the programmer and the ASH report
4. Generate explain plans (text file)


## Before you start..

### How to get the database id - DBID?
```sql
SELECT DBID FROM v$database;
```

### How to get the instance id - inst_id?
```sql
SELECT inst_id FROM gv$instance;
```

### How to get your sid?
```sql
SELECT SYS_CONTEXT ('USERENV', 'SID') FROM DUAL;
```

## Generate ASH report via SQL commands
Run the ash report using one of those: 
	- for time frame
```sql
select * from table(DBMS_WORKLOAD_REPOSITORY.ASH_REPORT_HTML(
  L_DBID =>     <db_id>,         -- this is number
  L_INST_NUM => <inst_id>,       -- this is number usualy 1
  L_BTIME => sysdate - 60/1440, /* meaning start the report from 60 minutes ago */
  L_ETIME => sysdate - 30/1440  /* Until 30 minutes ago */
  ));
```
### For an exact date:
You can Run ASH report for an exact date:
```sql
select * from table(DBMS_WORKLOAD_REPOSITORY.ASH_REPORT_HTML(
  L_DBID =>     <db_id>,     -- this is number
  L_INST_NUM => <inst_id>,   -- this is number usualy 1
  L_BTIME => TO_DATE('22/2/2013 06:20','dd/mm/yyyy hh24:mi'),
  L_ETIME => TO_DATE('22/2/2013 06:40','dd/mm/yyyy hh24:mi'),
  L_SQL_ID =>'4susktwfs9jg2'));

select * from table(DBMS_WORKLOAD_REPOSITORY.ASH_REPORT_HTML(
  L_DBID => 123456, 
  L_INST_NUM => 1,  
  L_BTIME => TO_DATE('23/2/2019 10:00','dd/mm/yyyy hh24:mi'),
  L_ETIME => TO_DATE('23/2/2019 16:00','dd/mm/yyyy hh24:mi'),
  L_WAIT_CLASS =>'Commit' ));
```
### Example for a client id
```sql
select * from table(DBMS_WORKLOAD_REPOSITORY.ASH_REPORT_HTML(
  L_DBID =>     62116283,         
  L_INST_NUM => 1,       -- this is number usualy 1
  L_BTIME => TO_DATE('11/02/2019 15:06:20','dd/mm/yyyy hh24:mi:ss'),
  L_ETIME => TO_DATE('11/02/2019 15:09:20','dd/mm/yyyy hh24:mi:ss'),
  L_client_id=>'YourClientId'
  ));
 
```

Copy the result into a text editor. Save with HTML extention, and there you have your AWR report.


## Lets view an execution plan 
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


### B. Create explain plan with actual run-time statistics
1. Run the query and gather statistics
    ```sql
    SELECT /*+ GATHER_PLAN_STATISTICS */  * 
    FROM stg_tst_interest_sem_um1_vu a;
    ```

2. Display the plan generated
#### Option 1 - For the last statement executed
Remark: *sometimes it does not work...*
```sql
SELECT * FROM TABLE (DBMS_XPLAN.DISPLAY_CURSORFORMAT=>'ALLSTATS LAST'));
```

#### Option 2 - For the last statement executed for a specific SQL ID
* (Use this if option 1 did no work)
a. Get the SQL ID of the statement
    ```sql
    SELECT sql_id, sql_text 
    FROM v$sqlarea 
    WHERE sql_text LIKE '%GATHER_PLAN_STATISTICS%' 
     and sql_text NOT LIKE '%sqlarea%';
    ```
b. Get execution plan with actual run-time statistics
    *(Replace the "&sql_id" with the actual SQL ID of the statement)*
```sql
    SELECT * 
    FROM TABLE (DBMS_XPLAN.DISPLAY_CURSOR(sql_id => '&sql_id',FORMAT=>'ALLSTATS LAST'));
```

### C. Create explain plan with actual run-time statistics using MONITOR report
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

## Generate AWR remotely via SQL Developer or SQL*Plus

Using: 
```sql
 select * 
 from table(DBMS_WORKLOAD_REPOSITORY.AWR_REPORT_HTML(
		L_DBID     => <DBID>, 
		L_INST_NUM => <INSTANCE_NUMBER>, 
		L_BID      => <BEGIN_SNAP>, 
		L_EID      => <END_SNAP>));
```

For example:
1. Check DB_ID:
```sql
SELECT DBID FROM v$database;
```

2. Check required snapshots:
```sql
select 
	s.snap_id AS "snap_id",  
	to_char(s.end_interval_time,'dd/mm/yyyy HH24:mi') AS "snapshot time",
	to_char(s.startup_time,'dd Mon "at" HH24:mi:ss')  AS "Startup time"
from dba_hist_snapshot s
order by snap_id DESC;
```
You should choose snap_id for begin () and end parameters

3. Generate a report for 9:00 to 10:00 today:
```sql 
select * 
from table(DBMS_WORKLOAD_REPOSITORY.AWR_REPORT_HTML(
        L_DBID        => 625123283, 
        L_INST_NUM    => 1, 
        L_BID         => 46980, 
        L_EID         => 46982));
```

4. Copy the result into notepad. Save with HTML extention, and there you have your AWR report.


