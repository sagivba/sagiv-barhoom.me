---
layout: post
title:  "DBMS_JOBS snippets"
author: "Sagiv Barhoom"
date:   2020-01-02
categories: ORACLE 
---

# ```DBMS_JOBS``` snippets
Here some basic snippets to mange ```user_jobs```/```dba_jobs```...
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
SELECT job, log_user, priv_user,schema_user, 
       last_date, next_date,what, broken, failures 
FROM dba_jobs j 
WHERE j.broken='Y'
```
## Run job 1864
```sql
DBMS_JOB.RUN(job=>1864,force=>FALSE);
```
* The force parameter default is FALSE
* If force is FALSE, the job can run in the foreground only in the specified instance. 
  Oracle displays error ```ORA-23428``` if force is FALSE and the connected instance is the incorrect instance.

## Fix broken job 1864
Run the package bellow as  *USER*:
```sql
EXEC DBMS_JOB.BROKEN(1864, FALSE);
COMMIT;
```

## Remove job 1864
```sql
BEGIN
   DBMS_JOB.REMOVE(1864);
   COMMIT;
END; 
```
