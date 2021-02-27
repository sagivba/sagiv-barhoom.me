---
layout: post
title:  "Analytic Functions - RANK() vs DENSE_RANK() vs ROW_NUMBER()"
author: "Sagiv Barhoom"
date:   2021-02-27
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---
# Analytic Functions - `RANK()` vs `DENSE_RANK() vs `ROW_NUMBER()`
This is a quick reminder - when to use each funcion.

```
[RANK()| DENSE_RANK()| ROW_NUMBER(])  OVER ([ query_partition_clause ] order_by_clause)
order by:
ORDER BY expression1 [,expression2,...] [ASC | DESC ] [NULLS FIRST | LAST]
```

### the `RANK()` function

The `RANK()` function calculates the rank of a value in a set of values.
It will skip rank in case of ties.
`[A A A B C D]  => [1 1 1 4 5 6]`


### the `DENSE_RANK()` function
Unlike the RANK() function, the DENSE_RANK() function returns rank values as consecutive integers.
It does not skip rank in case of ties.
Rows with the same values for the rank criteria will receive the same rank values.
`[A A A B C D]  => [1 1 1 2 3 4]`

DENSE_RANK( ) OVER([ query_partition_clause ] order_by_clause)

### the `ROW_NUMBER()` function
Assign a unique sequential integer starting from 1 to each row in a partition or in the whole result.
`[A A A B C D]  => [1 2 3 4 5 6]`

## An example
### the data
```sql
SELECT semester, course, class, student_id ,grade 
FROM student_grades
where semester='2015a'
```

| semester | course | class | student_id | grade  |
|----------|--------|-------|------------|--------|
| 2015a    | math101| A     |  st-001    |  95    |
| 2015a    | math101| A     |  st-002    |  95    |
| 2015a    | math101| A     |  st-003    |  80    |
| 2015a    | math101| A     |  st-004    |  60    |
| 2015a    | math101| A     |  st-005    |  60    |
| 2015a    | math101| B     |  st-011    | 100    |
| 2015a    | math101| B     |  st-012    |  70    |
| 2015a    | math101| B     |  st-013    |  70    |
| 2015a    | math101| B     |  st-014    |  70    |
| 2015a    | math101| B     |  st-015    |  62    |

### 
```sql
SELECT  class, student_id ,grade,
  RANK()       OVER( PARTITION BY semester, course, class ORDER BY grade DESC) AS rank,
  DENSE_RANK() OVER( PARTITION BY semester, course, class ORDER BY grade DESC) AS dense_rank,
  ROW_NUMBER() OVER( PARTITION BY semester, course, class ORDER BY grade DESC) AS row_number
FROM student_grades
where semester='2015a';
```


| class | student_id | grade  | rank | dense_rank | row_number |
|-------|------------|--------|------|------------|------------|
| A     |  st-001    |  95    |  1   |     1      |     1      |
| A     |  st-002    |  95    |  1   |     1      |     2      |
| A     |  st-003    |  80    |  3   |     2      |     3      |
| A     |  st-004    |  60    |  4   |     3      |     4      |
| A     |  st-005    |  60    |  4   |     3      |     5      |
| B     |  st-011    | 100    |  1   |     1      |     1      |
| B     |  st-012    |  70    |  2   |     2      |     2      |
| B     |  st-013    |  70    |  2   |     2      |     3      |
| B     |  st-014    |  70    |  2   |     3      |     4      |
| B     |  st-015    |  62    |  5   |     4      |     5      |


