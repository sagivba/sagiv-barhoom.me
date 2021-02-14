---
layout: post
title:  "Querying V$ACTIVE_SESSION_HISTORY"
author: "Sagiv Barhoom"
date:   2021-02-16
categories: ORACLE 
---

# Querying ```V$ACTIVE_SESSION_HISTORY```
```V$ACTIVE_SESSION_HISTORY``` displays sampled session activity in the database. 
It contains snapshots of active database sessions taken once a second.
Here are some snippest which I am using.

##  Take a look what run at a given minute
```sql
SELECT TO_CHAR (hs.sample_time, 'hh24:Mi') my_time, hs.*
FROM V$ACTIVE_SESSION_HISTORY hs
WHERE TO_CHAR (hs.sample_time, 'hh24:Mi') LIKE '00:0%'
ORDER BY 2;
```
## Distribution of activity by minute
```sql
SELECT TO_CHAR (hs.sample_time, 'hh24:Mi') my_time, COUNT (*) N
FROM V$ACTIVE_SESSION_HISTORY hs
GROUP BY TO_CHAR (hs.sample_time, 'hh24:Mi')
ORDER BY 1;
```
## The most common queries per time
for example get the in the most common queries of first three minutes of the day.
```sql
SELECT sql_id, COUNT (*)
FROM (  
     SELECT TO_CHAR (hs.sample_time, 'hh24:Mi') my_time, hs.*
     FROM V$ACTIVE_SESSION_HISTORY hs
     WHERE TO_CHAR (hs.sample_time, 'hh24:Mi') IN('00:01', '00:02', '00:03')
     ORDER BY 2
     )
GROUP BY sql_id
ORDER BY COUNT (*) DESC;
```
```sql
SELECT sql_id, module, machine, program, COUNT (*) N
FROM (  
    SELECT TO_CHAR (hs.sample_time, 'hh24:Mi') my_time, hs.*
    FROM V$ACTIVE_SESSION_HISTORY hs
    WHERE TO_CHAR (hs.sample_time, 'hh24:Mi') IN ('00:01', '00:02', '00:03')
    ORDER BY 2)
GROUP BY sql_id, module, machine, program, 
ORDER BY COUNT (*) DESC;
```

### show the sql text of the above queries

```sql
SELECT sql_text
       -- sql_fulltext
       -- *
FROM v$SQL
WHERE sql_id IN (SELECT ...      )
```


