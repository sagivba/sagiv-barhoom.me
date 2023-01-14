---   
layout: post
title: "Oracle DBMS_SCHEDULER 101"
subtitle: "Collecting info using explain-plan, ASH reprot and AWR report"
date: 2022-08-12
background: '/img/posts/engine.jpg'
categories: Oracle
--- 

# Oracle DBMS_SCHEDULER 101
A short how-to for Oracle `DBMS_SCHEDULER`
When you want to create a job on the database as simple as possible.​

## creating new job in 3 ateps

1. create a program
2. create or choose the schedule
3. create the job

### create a program
parameters:
- The `program` can be: `PLSQL_BLOCK`, `STORED_PROCEDURE` or `EXECUTABLE`
- The program_action` - The actual work that is done by the program
- The `enabled` - is flag which indicates if the program is enabled or not.  
                  the program accepts arguments, disable it until you define the arguments using: `dbms_scheduler.define_program_argument` 
```sql
BEGIN
  -- PL/SQL Block.
  DBMS_SCHEDULER.create_program (
    program_name   => 'program_check_status_of_dialer',
    program_type   => 'STORED_PROCEDURE',
    program_action => 'dialr.check_status_of_dialer';
    enabled        => TRUE,
    comments       => 'this will check the starus of dialer records and insert report records into the table dialer_daily_report');
/* -- here is an exaample of define_program_argumen 
  DBMS_SCHEDULER.define_program_argument(
      program_name      => 'check_status_of_dialer',
      argument_position =>  1,
      argument_name     => 'REPORT_TYPE'
      argument_type     => 'varvhar2',
      default_value     =>  'FULL'
  );
*/
END;
```
### create or choose the schedule
#### list all current existing schedules
```sql
SELECT *  FROM DBA_SCHEDULER_SCHEDULES
```
#### create new schedule
```sql
BEGIN
 DBMS_SCHEDULER.CREATE_SCHEDULE (
    schedule_name     => 'once_a_day',
    start_date        => SYSTIMESTAMP,
    -- end_date          => SYSTIMESTAMP + INTERVAL '30' day,
    repeat_interval   => 'FREQ=DAILY; INTERVAL=1;',
    comments          => 'once a day');
END;
```
### create the job
```sql
BEGIN
  DBMS_SCHEDULER.create_job (
   job_name            =>  daily_dialer_report,
   program_name        =>  program_check_status_of_dialer,
   start_date          =>  trunc(sysdate+1),
   enabled             =>  TRUE,
   comments            =>  'daily check the starus of dialer records and insert report records into the table');
   /*
   dbms_scheduler.set_job_argument_value(
      job_name          => 'daily_dialer_report',
      argument_position => 1,
      argument_value    => 'FULL');
  */
END;
```
##  changing the job
### enable disable
```sql
  execute dbms_scheduler.enable('dialer_user.daily_dialer_report');
  execute dbms_scheduler.disable('jialer_user.daily_dialer_report');
```
### drop job

```sql
declare
   l_job_exists number;
begin
   select count(*) into l_job_exists
     from user_scheduler_jobs
    where job_name = 'daily_dialer_report'
          ;

   if l_job_exists = 1 then
      dbms_scheduler.drop_job(job_name => 'daily_dialer_report');
   end if;
end;
```
### change job setings
```sql
    DBMS_SCHEDULER.SET_ATTRIBUTE
    ( NAME      => 'daily_dialer_report'
     ,attribute => 'AUTO_DROP'
     ,VALUE     => TRUE); -- this will make the job droped after running once
```
