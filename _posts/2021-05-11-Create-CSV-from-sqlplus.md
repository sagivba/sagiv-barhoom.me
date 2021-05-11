---
layout: post
title:  "Create CSV from sqlplus"
author: "Sagiv Barhoom"
date:   2021-05-11
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---

# Create csv from sqlplus
```sql
set colsep ,     -- separate columns with a comma
set pagesize 0   -- No header rows
set trimspool on -- remove trailing blanks
set linesize 200 -- 200 is the sum of the columns widths
set numwidth 12  -- 12 is the length for numbers 
set headsep off

spool students.csv

SELECT
  first_name||' '||last_name,
  id,
  avg_grade grade  
FROM
  stuednts_tbl
 WHERE first_year=2017;
 
 spool off
 ```
 
 
 READ THIS IN THE FUTURE:
 https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sqpug/SET-system-variable-summary.html#GUID-0AA910C4-C22A-4A9E-BE13-AAA059CC7919
