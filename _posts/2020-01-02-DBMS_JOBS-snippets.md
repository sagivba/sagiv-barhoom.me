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


## List running jobs
```sql
SELECT rj.job ,
       rj.sid,
       rj.failures,       
       Substr(To_Char(rj.last_date,'DD-Mon-YYYY HH24:MI:SS'),1,20) Last_Date,      
       Substr(To_Char(rj.this_date,'DD-Mon-YYYY HH24:MI:SS'),1,20) This_Date,
       j.what
FROM   dba_jobs_running rj  inner join dba_jobs j on rj.job=j.job
```



## export user jobs of one user
```sql
/* Formatted on 04/03/2021 19:11:25 (QP5 v5.326) */
DECLARE
    job_body    VARCHAR2 (32767):='--';
    job_id number;
    
BEGIN
    FOR record IN (SELECT JOB from dba_jobs where priv_user=USER  and job not in( 1360) order by JOB  )
    LOOP
        DBMS_JOB.user_export (record.job, job_body);
        DBMS_OUTPUT.put_line ('exec '|| job_body || '--' || record.job);
    END LOOP;
EXCEPTION
    WHEN OTHERS
    THEN
        RAISE;
END;
```
### output
```
dbms_job.isubmit(job=>43,what=>'PKG_1.B(''Y'');',next_date=>to_date('2021-03-04:04:20:06','YYYY-MM-DD:HH24:MI:SS'),interval=>'SYSDATE + 1/(24*12)',no_parse=>TRUE);--43
dbms_job.isubmit(job=>143,what=>'PKG_2.C;',next_date=>to_date('2021-03-04:04:22:06','YYYY-MM-DD:HH24:MI:SS'),interval=>'SYSDATE + 5/1440',no_parse=>TRUE);--14
```
