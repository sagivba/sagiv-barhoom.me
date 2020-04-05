---
layout: post
title:  "DBMS_JOBS snippets"
author: "Sagiv Barhoom"
date:   2020-01-02
categories: ORACLE,DBA 
---

# DBMS_JOBS snippets

## Create job
```sql
DECLARE
jobno NUMBER;
BEGIN
jobno:=-1;
DBMS_JOB.SUBMIT( 
   job       =>jobno,
   what      =>'YOUR_PKG.PROC;', 
   next_date =>		trunc(SYSDATE)+12.75/24,  -- date
   interval  =>     'trunc(SYSDATE+7)+15/24'  -- string
   );
dbms_output.put_line(jobno);   
END;
```

## Find broken jobs
As sys user run the example query bellow:
```sql
SELECT
 job, log_user, priv_user,schema_user, last_date, next_date,what, broken, failures 
SELECT * 
FROM dba_jobs j 
WHERE j.broken='Y'
```

## Fix broken job 1864
Run the package bellow as  *USER*:
```sql
EXEC dbms_job.broken(1864, FALSE);
commit;
```

## Remove job 1864
```sql
BEGIN
   DBMS_JOB.REMOVE(1864);
   COMMIT;
END; 
```
