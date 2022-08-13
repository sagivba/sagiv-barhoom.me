---
layout: post
title:  "Oracle and Parallelism"
author: "Sagiv Barhoom"
date:   2022-08-13
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---

# Oracle and Parallelism
Here are some notes on oracle parallelism in the Oracle DB

## Brief 
*DOP* - Degree Of Parallelism

Three ways to set DOP:
1. in a session - `ALTER SESSION SET parallel_degree_policy = auto;`
   then use hint `/*+ parallel(<DOP number fo process>) */`
2. setting a table or index  for parallelism
3. alter system - parameter PARALLEL_DEGREE_POLICY is set to AUTO


## Setting Automatic Degree of Parallelism Using ALTER SESSION Statements

We can set the DOP with two steps:
1. Use an ALTER SESSION statement 
2. Use the hint
`PARALLEL_DEGREE_POLICY` specifies whether the automatic degree of parallelism,
statement queuing, and in-memory parallel execution will be enabled.

```sql
ALTER SESSION SET parallel_degree_policy = auto;
```

### Setting Automatic Degree of Parallelism Using Hints
We can use the PARALLEL hint to force parallelism. 
It takes an optional parameter: the DOP at which the statement should run. 
In addition, the NO_PARALLEL hint overrides a `PARALLEL` parameter in the DDL that created or altered the table.
 - The following example illustrates forcing the statement to be executed in parallel:
```sql
SELECT /*+PARALLEL */ ename, dname FROM emp e, dept d WHERE e.deptno=d.deptno;
```

- The following example illustrates forcing the statement to be executed in parallel with a degree of 10:
```sql
SELECT /*+ parallel(10) */ ename, dname FROM emp e, dept d WHERE e.deptno=d.deptno;
```
- The following example illustrates forcing the statement to be executed serially:
```sql
SELECT /*+ no_parallel */ ename, dname FROM emp e, dept d WHERE e.deptno=d.deptno;
```

- We can also have multiple parallel hints within a query.  
  The following example shows two parallel hints within a query, one for each table::
```sql
select /*+ PARALLEL(employees 4) PARALLEL(dept 4) USE_HASH(emp) ORDERED */
   max(salary),
   avg(salary)
from  emp e inner inner join dept d on e.department_id = d.department_id
group by e.department_id;
```
In the above example, we use the USE_HASH to instruct the engine to use the hash method to join tables.
Oracle accesses the emp table and builds a hash table on the join key in memory.
It then scans the other table in the join.

- The following example illustrates computing the DOP the statement should use:
```sql
SELECT /*+ parallel(auto) */ ename, dname  FROM emp e, dept d
WHERE e.deptno=d.deptno;
```
## Specifying the Degree of Parallelism on table 
```sql
ALTER TABLE customers PARALLEL 4;
```
Here queries accessing the customers table request a DOP of 4.

## Automatic Parallel Degree Policy

When the parameter PARALLEL_DEGREE_POLICY is set to AUTO, 
Oracle Database automatically decides if a statement should execute in parallel or not and what DOP it should use.

For more information, see "Determining Degree of Parallelism" and "Controlling Automatic Degree of Parallelism" on Oracle documantation.

### The `PARALLEL_DEGREE_LIMIT` parameter 
This parameter sets a limit on the maximum DOP.  
The `PARALLEL_DEGREE_LIMIT` parameter acts as a governor to the PARALLEL hint, 
specifying the upper bound of parallelism for a parallelized query. 

The default for `PARALLEL_DEGREE_LIMIT` is cpu_count*2.

