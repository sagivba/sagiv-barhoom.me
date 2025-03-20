---
layout: post
title:  "SQLPlus COPY Intro"
author: "Sagiv Barhoom"
date:   2025-03-20
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---


# Optimizing SQL*Plus COPY for Large Table Transfers

We can use SQL*Plus COPY for transferring data between Oracle databases. 
It uses a detailed syntax. 
Example
```sql
COPY FROM my_user/password@source_db TO other_user/paswword@target_db  
REPLACE your_table -
USING SELECT id, name, degree FROM my_table;
```

## Performance Tweaks
Optimizing performance is crucial when transferring thousands or tens of thousands of records using `SQL*Plus COPY`. 
If there are no memory limitations, follow these best practices to speed up the process.

Fetching more rows per request reduces the number of round trips to the database.
```sql
SET ARRAYSIZE 5000   # Fetching more rows per request reduces the number of round trips to the database.
SET COPYCOMMIT 5000  # Committing after each batch prevents excessive undo and reduces locks.
# Disable unnecessary output may speed up execution
SET RECSEP OFF  
SET TERMOUT OFF  
SET FEEDBACK OFF
```
