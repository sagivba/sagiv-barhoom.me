---
layout: post
title:  " Type of partitioning in oracle 19c"
author: "Sagiv Barhoom"
date:   2022-12-31
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---

# Type of partitioning in oracle 19c
We should choose the type of partitioning that is best for our specific needs, and thats will depend on the characteristics of the data and the requirements of application. 
It is important to *carefully consider* the specific needs and requirements *before* deciding which type of partitioning to use.

In Oracle 19c, there are several types of partitioning options:

## Range partitioning
Range partitioning divides the data into ranges based on a partitioning key, such as a date or a numerical value. 
Each partition is assigned a specific range of values for the partitioning key.
When to use range pattitions:
1. When very large tables are frequently scanned by a range predicate on a good partitioning column (for example log date).
2. When we want to maintain a rolling window of data.
3. if we cannot complete administrative operations, such as backup and restore, on large tables in an allotted time frame, 
   we can try to divide those tables into partitions based on the partition range column.

An attempt to match new data for which there is no matching range partition would result in the following error:
```
ORA-14400: inserted partition key does not map to any partition 
```
We can avoid this, by using a MAXVALUE partition to catch everything that does not fall into previously existing range partitions.
```sql
CREATE TABLE log (
                  log_id INTEGER,
                  sevirity NUMBER, 
                  log_date DATE,
                  msg varchar2(2000)
PARTITION BY RANGE (log_date)
 (PARTITION logp01 VALUES LESS THAN (TO_DATE('2023-02-01', 'YYYY-MM-DD')),
  PARTITION logp02 VALUES LESS THAN (TO_DATE('2023-03-01', 'YYYY-MM-DD')),
  ...
  PARTITION logp12 VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD')),
  PARTITION logpN VALUES LESS THAN (MAXVALUE) 
  );
```
This is a poor example for the real world, since we should also create an automatic way which add our log partition every month 
(and thould consider to drop alod partitions every month)
a better solution can be implemented using *Interval partitioning*

## Hash partitioning
Hash partitioning divides the data into partitions based on a hash value of the partitioning key. 
The hash value is used to determine which partition the data belongs in.
## when shoul we use Hash partitions?
1. If we want to enable partial or full parallel partition-wise joins with likely equisized partitions.
2. To distribute data evenly among the nodes of an MPP platform that uses Oracle Real Application Clusters. 
   Consequently, we can minimize interconnect traffic when processing internode parallel statements.
3. when we use partition pruning and partition-wise joins according to a partitioning key that is mostly constrained by a distinct value or value list.
4. To randomly distribute data to avoid I/O bottlenecks if you do not use a storage management technique that stripes and mirrors across all available devices.

``sql 
CREATE TABLE students_exercises_in_course
  ( student_id      NUMBER,
    exercise_date   DATE, 
    course_code     VARCHAR2(6),
    semester        VARCHAR2(6),
    exercise_code   VARCHAR2(3),
    qustion_number  INTEGER,
    grade           INTEGER   
PARTITION BY HASH(student_id)
( PARTITION p1 TABLESPACE tbs1
, PARTITION p2 TABLESPACE tbs2
, PARTITION p3 TABLESPACE tbs3
, PARTITION p4 TABLESPACE tbs4
);
```
Here we created a table which is spreaded on 4 tablespaces wnd devides the data by th hash of the student_id colomn.
This partitioning asumes that most of our queries are on on students and not courses for example,


## List partitioning
List partitioning divides the data into partitions based on specific values of the partitioning key. 
You specify the specific values that belong in each partition.
The advantage of list partitioning is that you can group and organize unordered and unrelated sets of data in a natural way. 
The DEFAULT partition enables you to avoid specifying all possible value.
The following example is from oracle docs:
```sql
CREATE TABLE sales_by_region (item# INTEGER, qty INTEGER, 
             store_name VARCHAR(30), state_code VARCHAR(2),
             sale_date DATE)
     STORAGE(INITIAL 10K NEXT 20K) TABLESPACE tbs5 
     PARTITION BY LIST (state_code) 
     (
     PARTITION region_east
        VALUES ('MA','NY','CT','NH','ME','MD','VA','PA','NJ')
        STORAGE (INITIAL 8M) 
        TABLESPACE tbs8,
     PARTITION region_west
        VALUES ('CA','AZ','NM','OR','WA','UT','NV','CO')
        NOLOGGING,
     PARTITION region_south
        VALUES ('TX','KY','TN','LA','MS','AR','AL','GA'),
     PARTITION region_central 
        VALUES ('OH','ND','SD','MO','IL','MI','IA'),
     PARTITION region_null
        VALUES (NULL),
     PARTITION region_unknown
        VALUES (DEFAULT)
     );
  ```

## Composite partitioning
Composite partitioning combines two or more types of partitioning, such as range and hash partitioning, to divide the data into partitions.

## Interval partitioning
Interval partitioning is similar to range partitioning, *but it automatically creates new partitions* as needed when data is inserted that falls outside of the existing partition ranges.

```sql
CREATE TABLE log (
                  log_id INTEGER,
                  sevirity NUMBER, 
                  log_date DATE,
                  msg varchar2(2000)
PARTITION BY RANGE (log_date)
INTERVAL ( NUMTOYMINTERVAL(1, 'MONTH'))
 (PARTITION logp01 VALUES LESS THAN (TO_DATE('2023-02-01', 'YYYY-MM-DD')),
 )
```

## Reference partitioning
Reference partitioning allows you to partition a table based on *the values in a column of another table*.
The partitioning key is derived from the values in the referenced table.
in the example  above if we quering mainly on semester we can consider change the data structre so we will heve the tables:
 - students_per_semester
 - course_per_semester
 - exercises_in_course (without the semester)
 
In this case we can consider to use Reference partitioning
Take alook at the greate explaination by 
<iframe width="560" height="315" src="https://www.youtube.com/embed/NkUQfacSL38" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
