---
layout: post
title:  "Pivot Tables Using SQL"
author: "Sagiv Barhoom"
date:   2017-05-12
categories: ORACLE 
---

# Pivot-tables-using-sql

## Simple example
This is the basic syntax of the Oracle ```PIVOT``` clause:
```sql
SELECT
    select_list
FROM  table_name
PIVOT [XML] (
        pivot_clause
        pivot_for_clause
        pivot_in_clause
);
```

Example:
```sql
SELECT *
FROM courses_view
PIVOT(
   COUNT(course_id)
   FOR academic_desipline
   IN ( 'Law and Social' Law_and_Social,  -- this is an alias
        'Engineering',
        'Education',
        'Health'
      )
)
ORDER BY course_name;
```

## Lets add aggregaated colomns
We can use the ```pivot_clause`` to return the number of
```sql
SELECT *
FROM graduates_students_view
PIVOT(
   COUNT(student_id),
   AVG(grade)
   FOR academic_desipline
   IN ( 'Law and Social' Law_and_Social, 
        'Engineering',
        'Education',
        'Health'
      )
)
ORDER BY status;
```

## The ```pivot_in_clause```
We can write this clause as list as seen above, but  also as a subquery using the **```XML```** option - and then parse the result:
```sql
SELECT *
FROM graduates_students_view
PIVOT XML (
   COUNT(student_id),
   AVG(grade)
   FOR academic_desipline
   IN (
        SELECT REPLACE(desipline_name, ' ', '_')
        FROM academic_desiplines_view
      )
)
```


