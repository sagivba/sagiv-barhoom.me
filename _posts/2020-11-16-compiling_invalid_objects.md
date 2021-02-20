---
layout: post
title:  "Compiling Invalid Objects"
author: "Sagiv Barhoom"
date:   2020-11-16
categories: ORACLE 
---

# Compiling Invalid Objects
Some queries when compiling invalid objects



## Check invalid objects
```sql
-- total number of invalids
select count(*) from dba_objects o where o.STATUS='INVALID';
-- show all invalids
select * from dba_objects o where o.STATUS='INVALID';
-- show all invalids in schema WEBAPP
select * from dba_objects o where o.STATUS='INVALID' and o.OWNER='WEBAPP';
--  show all invalids with prefix "MY_"
select * from dba_objects o where o.STATUS='INVALID' and o.OBJECT_NAME like 'MY_%';
```

## compile all invalid objects using ```utlrp``` 
```bash
sqlplus / as sysdba $ORACLE_HOME/rdbms/admin/utlrp.sql
```

### monitoring utlrp while it is running 
```sql
-- Query returning the number of invalid objects remaining. This number should decrease with time.
SELECT COUNT(*) FROM sys.obj$ WHERE status IN (4, 5, 6);
-- Query returning the number of objects compiled so far. This number should increase with time.
SELECT COUNT(*) FROM sys.UTL_RECOMP_COMPILED;
-- Query showing jobs created by UTL_RECOMP
SELECT job_name FROM dba_scheduler_jobs WHERE job_name like 'UTL_RECOMP_SLAVE_%';
-- Query showing UTL_RECOMP jobs that are running
SELECT job_name FROM dba_scheduler_running_jobs   WHERE job_name like 'UTL_RECOMP_SLAVE_%';
```
## compile set of invalid objects using ```ALTER ... COMPILE```
```sql
SET head OFF;
SET echo OFF; 
SET termout OFF; 
--SET verify OFF; 
set pagesize 0;
spool _tmp_compile_invalids.sql;
--
SELECT 'ALTER PACKAGE ' || OWNER || '.' || object_name || ' COMPILE PACKAGE;'
           code
  FROM DBA_OBJECTS
 WHERE status = 'INVALID' AND object_type LIKE 'PACKAGE%'
UNION
SELECT 'ALTER SYNONYM ' || OWNER || '.' || object_name || ' COMPILE;'
           code
  FROM DBA_OBJECTS
 WHERE     status = 'INVALID'
       AND object_type LIKE 'SYNONYM%'
       AND owner <> 'PUBLIC'
UNION
SELECT 'ALTER PUBLIC SYNONYM ' || object_name || ' COMPILE;'     code
  FROM DBA_OBJECTS
 WHERE     status = 'INVALID'
       AND object_type LIKE 'SYNONYM%'
       AND owner = 'PUBLIC';
--
spool off;
SET echo ON;
SET termout ON; 
--
@_tmp_compile_invalids.sql;

exit;
```






