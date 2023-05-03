---
layout: post
title:  "Restore Deleted Packege Using Flashback Query"
author: "Sagiv Barhoom"
date:   2017-05-12
categories: ORACLE 
---

# Restore Deleted Packege Using Flashback Query
This post will help you to get source code of changed object using flashback query.

## using `AS OF timestamp` - the easy way
get code from 3 hours ago
```sql
SELECT text
  FROM dba_source AS OF timestamp to_timestamp(sysdate-3/24);                 --the SCN fro previos step
  WHERE owner = 'app_user' AND  name = 'INTERESTING_PKG'
  ORDER BY type,line; -- both pkg spec and body
```
## using `AS OF scn` 
### What is SCN
The system change number (SCN) is a logical, internal timestamp used by the Oracle Database. 
SCNs order events that occur within the database. 
The database uses SCNs to query and track changes. 
When a transaction commits, the database records a SCN for this commit.

### Get Current SCN
Get the current scn and timestamp:
```sql
SELECT 
    TO_CHAR (SYSDATE - 3 / 24, 'dd/mm/yyyy hh24:mi:ss')     time_of_SCN,
    TIMESTAMP_TO_SCN (SYSDATE - 3 / 24)                     SCN
FROM DUAL;
```

## Get the code

```sql
SELECT  text
FROM dba_source
as of scn 123456 --the SCN from the previos step
WHERE name='my_pkg';
```


