---
layout: post
title:  "LISTAGG - rows to delimited strings"
author: "Sagiv Barhoom"
date:   2022-11-25
categories: ORACLE 
background: ''
---

# listagg - rows to delimited strings

This is a little awkward that I did not know this before... 
But I came across the `LISTAGG` function for the first time yesterday. 
This function converts lines to a single string with a delimiter character. 
I find it useful when we want short strings (less than 4000 characters). 
If the length of the result is longer the 4000 charractera one can use the `ON OVERFLOW TRUNCATE '...' WITH COUNT` 
But I think you should try to avoid the usage of this function on such cases.

Here are some examples of its use:
we will use the table `emp` and `dept` form (`scott` schema)[https://www.orafaq.com/wiki/SCOTT]

## Tables for this demo
- `scott.emp`
- `scott.dept`
Quick reminder:
```sql
select *  from scott.dept order by 1;
```
the outpout:
```
    DEPTNO DNAME          LOC          
---------- -------------- -------------
        10 ACCOUNTING     NEW YORK     
        20 RESEARCH       DALLAS       
        30 SALES          CHICAGO      
        40 OPERATIONS     BOSTON        
```        
```sql
select * from scott.emp  order by deptno fetch first 6 rows only;
```
and the output:
```
     EMPNO ENAME      JOB              MGR HIREDATE         SAL       COMM     DEPTNO
---------- ---------- --------- ---------- --------- ---------- ---------- ----------
      7782 CLARK      MANAGER         7839 09-JUN-81       2450                    10
      7934 MILLER     CLERK           7782 23-JAN-82       1300                    10
      7839 KING       PRESIDENT            17-NOV-81       5000                    10
      7369 SMITH      CLERK           7902 17-DEC-80        800                    20
      7788 SCOTT      ANALYST         7566 09-DEC-82       3000                    20
      7566 JONES      MANAGER         7839 02-APR-81       2975                    20
      .... There are more records
```


## simple usage 
Get list of dielemted values as one line:
```sql    
select LISTAGG(dname, ',') csv_dnames from scott.dept;             --> 'ACCOUNTING,RESEARCH,SALES,OPERATIONS'

select '^('||listagg(dname, '|')||')$' csv_dnames from scott.dept; --> ^(ACCOUNTING|RESEARCH|SALES|OPERATIONS)$
```
when you suspect that the return string will be too long use  `ON OVERFLOW TRUNCATE`
To remove the overflow character count, you use the WITHOUT COUNT clause.
 from example ``ON OVERFLOW TRUNCATE '!!!TOO LONG!!!' WITHOUT COUNT`

for example:
```sql    
select LISTAGG(dname, ',' ON OVERFLOW TRUNCATE '...' ) csv_dnames from scott.dept;  --> will tuncate the end of the string and concateinate to it '...'

select LISTAGG(dname, ',' ON OVERFLOW TRUNCATE with count  ) csv_dnames from scott.dept; --> will add info on the number of the omitted values 
```

## jobs for each dept:
 
 - This is supported from Oracle 19C 
```sql
 select d.dname,   
       listagg (DISTINCT e.job,', ' on overflow truncate with count)  -- DISTINCT is supporter from 19C
                within group (order by e.job) jobs                    -- here we order the csv list by jobs 
  from scott.dept d INNER JOIN  scott.emp e ON d.deptno = e.deptno     
 group by d.dname;
```
The output: 
```
DNAME           JOBS
--------------- ----------------------------------------------------------------------------------------------------
ACCOUNTING      CLERK, MANAGER, PRESIDENT
RESEARCH        ANALYST, CLERK, MANAGER
SALES           CLERK, MANAGER, SALESMAN
```
### Notes:
- the output for each job is `DISTINCT`
- the jobs  are ןמ lexicographic order 





