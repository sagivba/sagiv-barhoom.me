---
layout: post
title:  " Conversion to partition table - oracle 19c"
author: "Sagiv Barhoom"
date:   2023-01-04
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---
# Covertion to partition table - oracle 19c

## online conversion to partition table:
A non-partitioned table can be converted to a partitioned table with a `MODIFY` clause added to the ALTER TABLE SQL statement.
In addition, the keyword `ONLINE` can be specified, enabling concurrent DML operations while the conversion is ongoing.</br>
*Also note for the `UPDATE INDEXES` clause*

Here is an example of online conversion:
```sql
ALTER TABLE students MODIFY
  PARTITION BY RANGE (student_id) INTERVAL (100)
  ( PARTITION P1 VALUES LESS THAN (100),
    PARTITION P2 VALUES LESS THAN (500)
   ) ONLINE
  UPDATE INDEXES
 ( IDX1_SALARY LOCAL,
   IDX2_EMP_ID GLOBAL PARTITION BY RANGE (employee_id)
  ( PARTITION IP1 VALUES LESS THAN (MAXVALUE))
 );
 ```

### The  `UPDATE INDEXES` clause (taken from Oracle documentation)
1. This clause can be used to change the partitioning state of indexes and storage properties of the indexes being converted.
2. The specification of the `UPDATE INDEXES` clause is optional.
   Indexes are maintained both for the online and offline conversion to a partitioned table.
3. This clause cannot change the columns on which the original list of indexes are defined.
4. This clause cannot change the uniqueness property of the index or any other index property.
5. If you do not specify the tablespace for any of the indexes, then the following tablespace defaults apply.
    - Local indexes after the conversion collocate with the table partition.
    - Global indexes after the conversion reside in the same tablespace of the original global index on the non-partitioned table.
6. If you do not specify the INDEXES clause or the INDEXES clause does not specify all the indexes on the original non-partitioned table, then the following default behavior applies for all unspecified indexes.
    - Global partitioned indexes remain the same and retain the original partitioning shape.
    - Non-prefixed indexes become global nonpartitioned indexes.
    - Prefixed indexes are converted to local partitioned indexes.
    - Prefixed means that the partition key columns are included in the index definition, 
      but the index definition is not limited to including the partitioning keys only.

    - Bitmap indexes become local partitioned indexes, regardless of whether they are prefixed or not.
      Bitmap indexes must always be local partitioned indexes.

*The conversion operation cannot be performed if there are domain indexes.*



## offline conversion to partition table:

1. Create a new partitioned table using the CREATE TABLE statement.
2. Use the SELECT INTO statement to insert the data from the non-partitioned table into the partitioned table.
3. Rename the original non-partitioned table to a temporary name using the ALTER TABLE RENAME TO statement.
4. Rename the new partitioned table to the original name using the ALTER TABLE RENAME TO statement.
5. Drop the temporary non-partitioned table.
6. Compile invalid objects if needed

```sql
-- 1) Create the new partitioned table
CREATE TABLE partitioned_table ( id NUMBER,  name VARCHAR2(50),  date_col DATE)
PARTITION BY RANGE (date_col)
(
  PARTITION p1 VALUES LESS THAN (TO_DATE('2022-01-01', 'YYYY-MM-DD')),
  PARTITION p2 VALUES LESS THAN (TO_DATE('2022-06-01', 'YYYY-MM-DD')),
  PARTITION p3 VALUES LESS THAN (TO_DATE('2022-12-01', 'YYYY-MM-DD')),
  PARTITION p4 VALUES LESS THAN (TO_DATE('2023-06-01', 'YYYY-MM-DD'))
);

--2. Insert the data from the non-partitioned table into the partitioned table
INSERT INTO partitioned_table SELECT * FROM non_partitioned_table;

--3. Rename the original non-partitioned table to a temporary name
ALTER TABLE non_partitioned_table RENAME TO temp_non_partitioned_table;

--4. Rename the new partitioned table to the original name
ALTER TABLE partitioned_table RENAME TO non_partitioned_table;

--5. Drop the temporary non-partitioned table
DROP TABLE temp_non_partitioned_table;
```
